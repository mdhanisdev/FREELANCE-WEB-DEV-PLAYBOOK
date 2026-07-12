# AWS-S3 SETUP [ DIRECT-TO-BUCKET UPLOADS / PRESIGNED URLS ]

------------------------------------------------------------------------

Amazon S3 is the industry-standard object store. The production pattern
is to generate a short-lived presigned PUT URL on your server, then have
the browser upload the file bytes DIRECTLY to S3. Your server never
proxies the file, so it stays cheap and fast. You persist only the
object key in your database.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
pnpm add uuid
pnpm add -D @types/uuid
```

Add credentials to `.env.local` (placeholders — replace with your own):

```bash
AWS_REGION="us-east-1"
AWS_ACCESS_KEY_ID="AKIA_PLACEHOLDER_KEY"
AWS_SECRET_ACCESS_KEY="placeholder_secret_replace_me"
S3_BUCKET_NAME="my-app-uploads"
```

------------------------------------------------------------------------

## STEP 2 : Create a Shared S3 Client

```ts
// src/lib/s3.ts
import { S3Client } from "@aws-sdk/client-s3";

export const s3 = new S3Client({
  region: process.env.AWS_REGION!,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});
```

------------------------------------------------------------------------

## STEP 3 : Presigned PUT URL Route

Authenticate, validate type/size, and rename the file with a UUID so
users cannot overwrite each other's objects or inject path segments.

```ts
// src/app/api/s3/presign/route.ts
import { NextResponse } from "next/server";
import { PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { v4 as uuid } from "uuid";
import { s3 } from "@/lib/s3";
import { auth } from "@/lib/auth";

const ALLOWED = ["image/png", "image/jpeg", "image/webp"];
const MAX_BYTES = 5 * 1024 * 1024; // 5 MB

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

  // Rename with UUID; never trust the client filename.
  const key = `uploads/${session.user.id}/${uuid()}`;

  const url = await getSignedUrl(
    s3,
    new PutObjectCommand({
      Bucket: process.env.S3_BUCKET_NAME!,
      Key: key,
      ContentType: contentType,
    }),
    { expiresIn: 60 } // 60 seconds
  );

  return NextResponse.json({ url, key });
}
```

------------------------------------------------------------------------

## STEP 4 : Client Uploads Directly to S3

```tsx
// src/app/dashboard/s3-upload.tsx
"use client";

import { useState } from "react";

export function S3Upload() {
  const [key, setKey] = useState<string | null>(null);

  async function handle(file: File) {
    const res = await fetch("/api/s3/presign", {
      method: "POST",
      body: JSON.stringify({ contentType: file.type, size: file.size }),
    });
    const { url, key } = await res.json();

    // Bytes go straight to S3 — this request never hits your server.
    await fetch(url, {
      method: "PUT",
      headers: { "Content-Type": file.type },
      body: file,
    });

    // Persist the returned key to your DB.
    await fetch("/api/media", {
      method: "POST",
      body: JSON.stringify({ key }),
    });
    setKey(key);
  }

  return (
    <input
      type="file"
      accept="image/png,image/jpeg,image/webp"
      onChange={(e) => e.target.files?.[0] && handle(e.target.files[0])}
    />
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Upload Flow

```text upload-flow
  Browser              Next.js Server               AWS S3
     |                       |                          |
     |  1. POST /presign     |  auth + validate         |
     |  {type,size}          |  type/size, uuid key     |
     |---------------------->|                          |
     |                       |  2. sign PutObject       |
     |  3. {url, key}        |                          |
     |<----------------------|                          |
     |                                                  |
     |  4. PUT file bytes to presigned URL (direct)     |
     |------------------------------------------------->|
     |<-------------------------------------------------|
     |                       |                          |
     |  5. POST key to /api/media -> save in DB          |
     |---------------------->|                          |
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Sign on the server; never expose AWS keys to the browser
✓ Authenticate the user before issuing any presigned URL
✓ Validate contentType against an allowlist and cap the size
✓ Rename every object to a UUID key scoped by user id
✓ Set a short expiresIn (30-60s) so URLs cannot be reused
✓ Store only the S3 key in the DB; build URLs on demand
✓ Serve reads via CloudFront + presigned GET, keep bucket private
✓ Enforce a bucket CORS policy that allows only your origin
✓ Add a server-side size guard; the client value is a hint only
```
