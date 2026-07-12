# CLOUDFLARE-R2 SETUP [ S3-COMPATIBLE / ZERO EGRESS FEES ]

------------------------------------------------------------------------

Cloudflare R2 is an S3-compatible object store with ZERO egress fees.
You keep the exact AWS SDK workflow — presigned PUT URLs, direct browser
uploads — but pay nothing to serve bytes out. Choose R2 when you have
high read/download traffic (public assets, video, large downloads) and
S3 egress bills would dominate your costs.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

R2 speaks the S3 API, so the same AWS SDK packages apply.

```bash
pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner uuid
pnpm add -D @types/uuid
```

Add credentials to `.env.local` (placeholders — replace with your own):

```bash
R2_ACCOUNT_ID="placeholder_account_id"
R2_ACCESS_KEY_ID="placeholder_access_key"
R2_SECRET_ACCESS_KEY="placeholder_secret_replace_me"
R2_BUCKET_NAME="my-app-uploads"
R2_PUBLIC_BASE_URL="https://cdn.example.com"
```

------------------------------------------------------------------------

## STEP 2 : Create the R2 Client

The only differences from S3: `region: "auto"` and a custom `endpoint`
pointing at your account's R2 URL.

```ts
// src/lib/r2.ts
import { S3Client } from "@aws-sdk/client-s3";

export const r2 = new S3Client({
  region: "auto",
  endpoint: `https://${process.env.R2_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
});
```

------------------------------------------------------------------------

## STEP 3 : Presigned PUT URL Route

Authenticate, validate type and size, and rename to a UUID key.

```ts
// src/app/api/r2/presign/route.ts
import { NextResponse } from "next/server";
import { PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { v4 as uuid } from "uuid";
import { r2 } from "@/lib/r2";
import { auth } from "@/lib/auth";

const ALLOWED = ["image/png", "image/jpeg", "image/webp", "application/pdf"];
const MAX_BYTES = 10 * 1024 * 1024; // 10 MB

export async function POST(req: Request) {
  const session = await auth();
  if (!session?.user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const { contentType, size } = await req.json();
  if (!ALLOWED.includes(contentType)) {
    return NextResponse.json({ error: "Bad file type" }, { status: 400 });
  }
  if (typeof size !== "number" || size > MAX_BYTES) {
    return NextResponse.json({ error: "File too large" }, { status: 400 });
  }

  const key = `uploads/${session.user.id}/${uuid()}`;

  const url = await getSignedUrl(
    r2,
    new PutObjectCommand({
      Bucket: process.env.R2_BUCKET_NAME!,
      Key: key,
      ContentType: contentType,
    }),
    { expiresIn: 60 }
  );

  return NextResponse.json({ url, key });
}
```

------------------------------------------------------------------------

## STEP 4 : Client Uploads Directly to R2

```tsx
// src/app/dashboard/r2-upload.tsx
"use client";

export function R2Upload() {
  async function handle(file: File) {
    const { url, key } = await (await fetch("/api/r2/presign", {
      method: "POST",
      body: JSON.stringify({ contentType: file.type, size: file.size }),
    })).json();

    // Bytes go straight to R2 — never through your server.
    await fetch(url, {
      method: "PUT",
      headers: { "Content-Type": file.type },
      body: file,
    });

    // Persist the key; build a public URL from R2_PUBLIC_BASE_URL.
    await fetch("/api/media", {
      method: "POST",
      body: JSON.stringify({ key }),
    });
  }

  return (
    <input type="file" accept="image/png,image/jpeg,image/webp,application/pdf"
      onChange={(e) => e.target.files?.[0] && handle(e.target.files[0])} />
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Upload Flow

```text upload-flow
  Browser             Next.js Server            Cloudflare R2
     |                      |                         |
     |  1. POST /presign    |  auth + validate        |
     |  {type,size}         |  type/size, uuid key    |
     |--------------------->|  2. sign PutObject      |
     |  3. {url, key}       |     (region: auto)      |
     |<---------------------|                         |
     |                                                |
     |  4. PUT file bytes to presigned URL (direct)   |
     |----------------------------------------------->|
     |<-----------------------------------------------|
     |                      |                         |
     |  5. save key in DB; read via public CDN URL    |
     |     (zero egress fees on delivery)             |
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Reuse the AWS SDK — R2 is S3-compatible, only endpoint changes
✓ Set region "auto" and the account-scoped R2 endpoint
✓ Authenticate before issuing any presigned URL
✓ Validate contentType allowlist and cap size server-side
✓ Rename every object to a UUID key scoped by user id
✓ Keep expiresIn short (30-60s) so URLs cannot be reused
✓ Store only the key; build public URLs from R2_PUBLIC_BASE_URL
✓ Choose R2 for download-heavy apps to avoid S3 egress bills
✓ Bind a custom domain and cache with Cloudflare for free delivery
```
