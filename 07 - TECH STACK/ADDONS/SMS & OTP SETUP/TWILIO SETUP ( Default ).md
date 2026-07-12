# TWILIO SETUP [ SMS & OTP ]

------------------------------------------------------------------------

Twilio Verify handles one-time-passcode delivery and validation for you:
you request a code for a phone number, Twilio sends the SMS, and you ask
Twilio to check the code the user typed. No code storage on your side.

------------------------------------------------------------------------

## STEP 1 : Provision & Configure

Create a Twilio account, then create a **Verify Service** in the console.
Collect your Account SID, Auth Token, and the Verify Service SID.

```bash
pnpm add twilio
```

```text
.env.local
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_placeholder
TWILIO_VERIFY_SERVICE_SID=VAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

------------------------------------------------------------------------

## STEP 2 : Create A Shared Client

```ts
// lib/twilio.ts
import twilio from "twilio";

export const client = twilio(
  process.env.TWILIO_ACCOUNT_SID!,
  process.env.TWILIO_AUTH_TOKEN!,
);

export const VERIFY_SID = process.env.TWILIO_VERIFY_SERVICE_SID!;
```

------------------------------------------------------------------------

## STEP 3 : Send An OTP

```ts
// app/api/otp/send/route.ts
import { NextRequest, NextResponse } from "next/server";
import { client, VERIFY_SID } from "@/lib/twilio";

export async function POST(req: NextRequest) {
  const { phone } = await req.json(); // E.164, e.g. +15555550123
  await client.verify.v2
    .services(VERIFY_SID)
    .verifications.create({ to: phone, channel: "sms" });

  return NextResponse.json({ sent: true });
}
```

------------------------------------------------------------------------

## STEP 4 : Verify The OTP

```ts
// app/api/otp/check/route.ts
import { NextRequest, NextResponse } from "next/server";
import { client, VERIFY_SID } from "@/lib/twilio";

export async function POST(req: NextRequest) {
  const { phone, code } = await req.json();
  const result = await client.verify.v2
    .services(VERIFY_SID)
    .verificationChecks.create({ to: phone, code });

  if (result.status !== "approved") {
    return NextResponse.json({ error: "invalid code" }, { status: 401 });
  }
  return NextResponse.json({ verified: true });
}
```

------------------------------------------------------------------------

## STEP 5 : OTP Flow

```text
  User enters phone
        |
        v
  POST /api/otp/send ---> Twilio Verify ---> SMS to +1555...0123
        |
  User reads code, submits
        |
        v
  POST /api/otp/check --> verificationChecks --> status:approved?
        yes -> issue session / continue
        no  -> 401 invalid code
```

------------------------------------------------------------------------

## STEP 6 : Windows Notes

Runs identically under `pnpm dev` on Windows. If you use the Twilio CLI
for local webhook testing, install it via `scoop install twilio` or the
Windows MSI rather than the macOS Homebrew path. Keep the Auth Token out
of source control and avoid CRLF in `.env.local`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always store phone numbers in E.164 format (+15555550123)
✓ Use Verify service, not raw SMS, so codes never touch your DB
✓ Keep Account SID + Auth Token server-only
✓ Rate-limit send attempts per phone and per IP
✓ Set a max-attempts policy; Verify auto-expires codes (~10 min)
✓ Never log the OTP code or full phone number
✓ Handle carrier failures gracefully and offer a resend button
```
