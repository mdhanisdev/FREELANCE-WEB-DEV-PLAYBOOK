# AWS SNS SETUP [ SMS & OTP ]

------------------------------------------------------------------------

AWS SNS sends transactional SMS but does NOT manage OTP lifecycle. You
generate the code, deliver it via SNS `Publish`, and store a hashed copy
with a TTL so you can verify it yourself.

------------------------------------------------------------------------

## STEP 1 : IAM & Region Setup

Create an IAM user (or role) with `sns:Publish`, note the region, and
create programmatic credentials.

```bash
pnpm add @aws-sdk/client-sns
```

```text
.env.local
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=your_secret_access_key_placeholder
OTP_HASH_SECRET=replace_with_long_random_string
```

------------------------------------------------------------------------

## STEP 2 : Create The SNS Client

```ts
// lib/sns.ts
import { SNSClient, PublishCommand } from "@aws-sdk/client-sns";

export const sns = new SNSClient({ region: process.env.AWS_REGION });

export async function sendSms(phone: string, message: string) {
  await sns.send(
    new PublishCommand({
      PhoneNumber: phone, // E.164, e.g. +15555550123
      Message: message,
      MessageAttributes: {
        "AWS.SNS.SMS.SMSType": {
          DataType: "String",
          StringValue: "Transactional",
        },
      },
    }),
  );
}
```

------------------------------------------------------------------------

## STEP 3 : Generate & Send OTP

Hash the code before storing it (Redis with a TTL is a good fit).

```ts
// app/api/otp/send/route.ts
import { NextRequest, NextResponse } from "next/server";
import { createHmac, randomInt } from "crypto";
import { sendSms } from "@/lib/sns";
import { redis } from "@/lib/redis";

const hash = (code: string) =>
  createHmac("sha256", process.env.OTP_HASH_SECRET!).update(code).digest("hex");

export async function POST(req: NextRequest) {
  const { phone } = await req.json();
  const code = String(randomInt(100000, 999999));

  await redis.set(`otp:${phone}`, hash(code), { ex: 300 }); // 5 min TTL
  await sendSms(phone, `Your code is ${code}`);

  return NextResponse.json({ sent: true });
}
```

------------------------------------------------------------------------

## STEP 4 : Verify OTP

```ts
// app/api/otp/check/route.ts
import { NextRequest, NextResponse } from "next/server";
import { createHmac } from "crypto";
import { redis } from "@/lib/redis";

const hash = (code: string) =>
  createHmac("sha256", process.env.OTP_HASH_SECRET!).update(code).digest("hex");

export async function POST(req: NextRequest) {
  const { phone, code } = await req.json();
  const stored = await redis.get<string>(`otp:${phone}`);

  if (!stored || stored !== hash(code)) {
    return NextResponse.json({ error: "invalid code" }, { status: 401 });
  }
  await redis.del(`otp:${phone}`);
  return NextResponse.json({ verified: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Ownership Flow

```text
  Your server                     AWS SNS
  -----------                     -------
  gen 6-digit code
  store hash(code) TTL 5m
  Publish(phone, msg) --------->  carrier -> SMS
  ...user submits code...
  hash(input) == stored ?
        yes -> delete key, verified
        no  -> 401
```

Unlike Twilio Verify or Vonage Verify, YOU own generation, storage, and
expiry — plan for that extra responsibility.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`pnpm dev` is unaffected by OS. New AWS accounts start in the SMS sandbox
and can only message verified numbers until you request production access
and (in some regions) register a sender ID. The AWS SDK is pure JS, so no
native toolchain is needed on Windows.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Store only a hashed OTP with a short TTL, never plaintext
✓ Use randomInt (CSPRNG), not Math.random, for codes
✓ Set SMSType to Transactional for OTP delivery priority
✓ Delete the key on success to enforce single use
✓ Rate-limit sends per phone/IP to control spend
✓ Move out of the SNS sandbox before launch
✓ Never log the code or full phone number
```
