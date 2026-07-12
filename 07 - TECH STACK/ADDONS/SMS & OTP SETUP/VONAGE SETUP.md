# VONAGE SETUP [ SMS & OTP ]

------------------------------------------------------------------------

Vonage (formerly Nexmo) provides a Verify API that sends and checks OTP
codes, plus a raw SMS API. You start a verification, Vonage returns a
`request_id`, and you check the user's code against that request.

------------------------------------------------------------------------

## STEP 1 : Get API Credentials

Sign up at Vonage, then copy your **API Key** and **API Secret** from the
dashboard.

```bash
pnpm add @vonage/server-sdk @vonage/auth
```

```text
.env.local
VONAGE_API_KEY=xxxxxxxx
VONAGE_API_SECRET=your_api_secret_placeholder
VONAGE_BRAND_NAME=ExampleApp
```

------------------------------------------------------------------------

## STEP 2 : Create A Shared Client

```ts
// lib/vonage.ts
import { Vonage } from "@vonage/server-sdk";
import { Auth } from "@vonage/auth";

export const vonage = new Vonage(
  new Auth({
    apiKey: process.env.VONAGE_API_KEY!,
    apiSecret: process.env.VONAGE_API_SECRET!,
  }),
);
```

------------------------------------------------------------------------

## STEP 3 : Start A Verification

```ts
// app/api/otp/send/route.ts
import { NextRequest, NextResponse } from "next/server";
import { vonage } from "@/lib/vonage";

export async function POST(req: NextRequest) {
  const { phone } = await req.json(); // E.164 without leading +
  const resp = await vonage.verify.start({
    number: phone,
    brand: process.env.VONAGE_BRAND_NAME!,
  });

  return NextResponse.json({ requestId: resp.requestId });
}
```

------------------------------------------------------------------------

## STEP 4 : Check The Code

```ts
// app/api/otp/check/route.ts
import { NextRequest, NextResponse } from "next/server";
import { vonage } from "@/lib/vonage";

export async function POST(req: NextRequest) {
  const { requestId, code } = await req.json();
  const resp = await vonage.verify.check(requestId, code);

  if (resp.status !== "0") {
    return NextResponse.json({ error: "invalid code" }, { status: 401 });
  }
  return NextResponse.json({ verified: true });
}
```

------------------------------------------------------------------------

## STEP 5 : Verify Flow

```text
  Client                     Vonage Verify
  ------                     -------------
  POST /api/otp/send ----->  verify.start(number, brand)
        <----- request_id
  (user receives SMS code)
  POST /api/otp/check ---->  verify.check(request_id, code)
        status "0"  -> verified
        status !=0  -> 401
```

Persist `request_id` in a short-lived, HTTP-only cookie or session so the
check step can reference it. Status `"0"` means success in the Verify API.

------------------------------------------------------------------------

## STEP 6 : Windows Notes

`pnpm dev` works the same on Windows. The Vonage SDK is pure JavaScript,
so there are no native build steps to worry about. Keep secrets out of
version control and strip any CRLF from `.env.local` values.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Store the request_id server-side; never expose the API secret
✓ Send numbers in E.164 (Vonage accepts without the leading +)
✓ Cancel stale verifications before starting a new one per number
✓ Rate-limit start requests to control spend and abuse
✓ Map Vonage status codes explicitly; "0" is success
✓ Never log full phone numbers or codes
✓ Set a resend cooldown and a max-check-attempts guard
```
