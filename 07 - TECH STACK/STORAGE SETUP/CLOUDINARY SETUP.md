# CLOUDINARY SETUP [ IMAGE + VIDEO / MEDIA-HEAVY APPS ]

------------------------------------------------------------------------

Cloudinary is a media platform, not just storage. It stores images and
video AND transforms them on the fly — resize, crop, format conversion
(WebP / AVIF), and quality optimization via URL parameters. Choose it
when your app is media-heavy and you want automatic optimization and a
delivery CDN without building an image pipeline yourself.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add cloudinary next-cloudinary
```

Add credentials to `.env.local` (placeholders — replace with your own):

```bash
CLOUDINARY_CLOUD_NAME="my_cloud_placeholder"
CLOUDINARY_API_KEY="000000000000000"
CLOUDINARY_API_SECRET="placeholder_secret_replace_me"
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME="my_cloud_placeholder"
```

------------------------------------------------------------------------

## STEP 2 : Configure the Server SDK

```ts
// src/lib/cloudinary.ts
import { v2 as cloudinary } from "cloudinary";

cloudinary.config({
  cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
  api_key: process.env.CLOUDINARY_API_KEY,
  api_secret: process.env.CLOUDINARY_API_SECRET,
  secure: true,
});

export { cloudinary };
```

------------------------------------------------------------------------

## STEP 3 : Signed Upload Route

Sign the upload on your server so the API secret never reaches the
browser. Authenticate first and pin the params you will allow — the
signature must cover the exact params the client sends.

```ts
// src/app/api/cloudinary/sign/route.ts
import { NextResponse } from "next/server";
import { cloudinary } from "@/lib/cloudinary";
import { auth } from "@/lib/auth";
import { v4 as uuid } from "uuid";

export async function POST() {
  const session = await auth();
  if (!session?.user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  const timestamp = Math.round(Date.now() / 1000);
  const folder = `users/${session.user.id}`;
  const public_id = uuid(); // rename — do not trust client filename

  const signature = cloudinary.utils.api_sign_request(
    { timestamp, folder, public_id },
    process.env.CLOUDINARY_API_SECRET!
  );

  return NextResponse.json({
    signature,
    timestamp,
    folder,
    public_id,
    apiKey: process.env.CLOUDINARY_API_KEY,
    cloudName: process.env.CLOUDINARY_CLOUD_NAME,
  });
}
```

------------------------------------------------------------------------

## STEP 4 : Client Uploads Directly to Cloudinary

```tsx
// src/app/dashboard/cloudinary-upload.tsx
"use client";

const ALLOWED = ["image/png", "image/jpeg", "image/webp", "video/mp4"];
const MAX_BYTES = 20 * 1024 * 1024; // 20 MB

export function CloudinaryUpload() {
  async function handle(file: File) {
    if (!ALLOWED.includes(file.type)) return alert("Bad file type");
    if (file.size > MAX_BYTES) return alert("File too large");

    const s = await (await fetch("/api/cloudinary/sign", {
      method: "POST",
    })).json();

    const form = new FormData();
    form.append("file", file);
    form.append("api_key", s.apiKey);
    form.append("timestamp", s.timestamp);
    form.append("signature", s.signature);
    form.append("folder", s.folder);
    form.append("public_id", s.public_id);

    // Uploads straight to Cloudinary — skips your server entirely.
    const res = await fetch(
      `https://api.cloudinary.com/v1_1/${s.cloudName}/auto/upload`,
      { method: "POST", body: form }
    );
    const data = await res.json();
    // Persist data.public_id to your DB.
    console.log("public_id:", data.public_id);
  }

  return (
    <input type="file" accept="image/*,video/mp4"
      onChange={(e) => e.target.files?.[0] && handle(e.target.files[0])} />
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Transformations & Optimization

`f_auto` picks the best format (AVIF/WebP), `q_auto` optimizes quality.
`<CldImage>` from next-cloudinary applies these automatically.

```tsx
// src/app/gallery/photo.tsx
import { CldImage } from "next-cloudinary";

export function Photo({ id }: { id: string }) {
  return (
    <CldImage
      src={id}          // stored public_id
      width={800}
      height={600}
      crop="fill"
      gravity="auto"
      format="auto"     // f_auto
      quality="auto"    // q_auto
      alt="User upload"
    />
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Upload Flow

```text upload-flow
  Browser             Next.js Server            Cloudinary
     |                      |                        |
     |  1. POST /sign       |  auth + build params   |
     |--------------------->|  api_sign_request()    |
     |  2. {signature,...}  |                        |
     |<---------------------|                        |
     |                                               |
     |  3. POST file + signed params (direct)        |
     |---------------------------------------------->|
     |<----------------------------------------------|
     |     {public_id, secure_url}                   |
     |                                               |
     |  4. save public_id in DB --> render CldImage  |
     |     with f_auto / q_auto transforms           |
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Sign uploads server-side; API secret never touches the browser
✓ Authenticate the user before returning a signature
✓ Validate MIME type and size on the client AND in an upload preset
✓ Rename to a UUID public_id inside a per-user folder
✓ Store the public_id, not the full URL — URLs are derived
✓ Deliver with f_auto + q_auto for automatic optimization
✓ Use gravity=auto + crop=fill for responsive, subject-aware crops
✓ Choose Cloudinary for media-heavy apps needing transforms
✓ Set an unsigned-upload preset only for public, low-risk uploads
```
