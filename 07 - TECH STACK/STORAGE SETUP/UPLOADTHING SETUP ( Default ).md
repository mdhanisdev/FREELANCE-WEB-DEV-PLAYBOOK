# UPLOADTHING SETUP [ FILE UPLOADS / MANAGED STORAGE ]

------------------------------------------------------------------------

UploadThing is a managed upload service that handles storage, CDN, and
signing for you. It is the fastest way to ship file uploads in a Next.js
App Router project without provisioning a bucket yourself. You define a
type-safe file router, guard it with auth, and drop in generated React
components.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add uploadthing @uploadthing/react
```

Create a `.env.local` in the project root. Never commit this file.

```bash
UPLOADTHING_TOKEN="ut_placeholder_token_replace_me"
```

------------------------------------------------------------------------

## STEP 2 : Define the File Router (auth middleware)

The `middleware` runs on your server BEFORE the signed URL is issued.
Authenticate here and reject unauthenticated users. Validate the file
type and size via the route config.

```ts
// src/app/api/uploadthing/core.ts
import { createUploadthing, type FileRouter } from "uploadthing/next";
import { UploadThingError } from "uploadthing/server";
import { auth } from "@/lib/auth";

const f = createUploadthing();

export const ourFileRouter = {
  imageUploader: f({
    image: { maxFileSize: "4MB", maxFileCount: 1 },
  })
    // 1. Authenticate BEFORE any upload is allowed.
    .middleware(async () => {
      const session = await auth();
      if (!session?.user) throw new UploadThingError("Unauthorized");
      return { userId: session.user.id };
    })
    // 2. Runs on your server AFTER upload completes.
    .onUploadComplete(async ({ metadata, file }) => {
      console.log("Uploaded by:", metadata.userId);
      console.log("File key:", file.key, "URL:", file.ufsUrl);
      // Persist file.key + file.ufsUrl to your database here.
      return { uploadedBy: metadata.userId };
    }),
} satisfies FileRouter;

export type OurFileRouter = typeof ourFileRouter;
```

------------------------------------------------------------------------

## STEP 3 : Mount the Route Handler

```ts
// src/app/api/uploadthing/route.ts
import { createRouteHandler } from "uploadthing/next";
import { ourFileRouter } from "./core";

export const { GET, POST } = createRouteHandler({
  router: ourFileRouter,
});
```

------------------------------------------------------------------------

## STEP 4 : Generate Typed Components

```ts
// src/lib/uploadthing.ts
import {
  generateUploadButton,
  generateUploadDropzone,
} from "@uploadthing/react";
import type { OurFileRouter } from "@/app/api/uploadthing/core";

export const UploadButton = generateUploadButton<OurFileRouter>();
export const UploadDropzone = generateUploadDropzone<OurFileRouter>();
```

------------------------------------------------------------------------

## STEP 5 : Use in a Component

```tsx
// src/app/dashboard/upload-widget.tsx
"use client";

import { UploadButton, UploadDropzone } from "@/lib/uploadthing";

export function UploadWidget() {
  return (
    <div className="space-y-4">
      <UploadButton
        endpoint="imageUploader"
        onClientUploadComplete={(res) => {
          console.log("Files:", res);
        }}
        onUploadError={(e: Error) => alert(`Upload failed: ${e.message}`)}
      />
      <UploadDropzone
        endpoint="imageUploader"
        onClientUploadComplete={(res) => console.log("Done:", res)}
        onUploadError={(e: Error) => alert(e.message)}
      />
    </div>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Upload Flow

```text upload-flow
  Browser                Next.js Server           UploadThing
     |                         |                        |
     |  1. request upload      |                        |
     |------------------------>|  auth() middleware     |
     |                         |  validate type/size    |
     |                         |  2. get signed URL     |
     |                         |----------------------->|
     |                         |<-----------------------|
     |  3. presigned URL       |                        |
     |<------------------------|                        |
     |                                                  |
     |  4. PUT file directly (skips your server)        |
     |------------------------------------------------->|
     |                                                  |
     |             5. onUploadComplete (webhook) ------>|
     |                  save key + url to your DB       |
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Authenticate inside .middleware() before any URL is signed
✓ Constrain maxFileSize and maxFileCount per route
✓ Restrict accepted MIME types (image / pdf) in the route config
✓ Persist file.key and file.ufsUrl in your DB, not just the URL
✓ Throw UploadThingError for clean, typed client error messages
✓ Keep UPLOADTHING_TOKEN in .env.local, never in the client bundle
✓ Bytes go browser -> UploadThing directly, never through your server
✓ Delete orphaned files via the server SDK when a record is removed
```
