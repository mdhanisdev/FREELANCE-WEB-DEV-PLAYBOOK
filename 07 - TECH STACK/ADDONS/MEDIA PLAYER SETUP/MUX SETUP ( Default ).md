# MUX SETUP [ MEDIA PLAYER ]
------------------------------------------------------------------------

Mux handles video ingest, transcoding, adaptive-bitrate (HLS) streaming,
and playback analytics. You upload a source file, Mux returns a playback
ID, and the `<MuxPlayer>` element streams the right quality for each
viewer. This is the recommended default for hosted video.

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add @mux/mux-player-react @mux/mux-node
```

`@mux/mux-player-react` is the client player; `@mux/mux-node` is the
server SDK for creating uploads and signing tokens.

------------------------------------------------------------------------

## STEP 2 : Configure Environment

Create an access token in the Mux dashboard (Settings -> Access Tokens).

```bash
# .env.local
MUX_TOKEN_ID=your_token_id
MUX_TOKEN_SECRET=your_token_secret
```

> Windows note: keep secrets in `.env.local`; do not paste tokens into
> PowerShell history. Next.js loads the file automatically on `pnpm dev`.

------------------------------------------------------------------------

## STEP 3 : Create a Direct Upload (Server)

```ts
// app/api/mux/upload/route.ts
import Mux from "@mux/mux-node";

const mux = new Mux({
  tokenId: process.env.MUX_TOKEN_ID!,
  tokenSecret: process.env.MUX_TOKEN_SECRET!,
});

export async function POST() {
  const upload = await mux.video.uploads.create({
    cors_origin: "*",
    new_asset_settings: { playback_policy: ["public"] },
  });
  return Response.json({ url: upload.url, id: upload.id });
}
```

The browser PUTs the file straight to `upload.url`; your server never
proxies the bytes.

------------------------------------------------------------------------

## STEP 4 : Render the Player (Client)

```tsx
// components/VideoPlayer.tsx
"use client";
import MuxPlayer from "@mux/mux-player-react";

export function VideoPlayer({ playbackId }: { playbackId: string }) {
  return (
    <MuxPlayer
      playbackId={playbackId}
      streamType="on-demand"
      accentColor="#000000"
      className="aspect-video w-full rounded-lg"
      metadata={{ video_title: "Demo" }}
    />
  );
}
```

The player auto-selects HLS renditions and reports Mux Data metrics.

------------------------------------------------------------------------

## STEP 5 : Ingest and Playback Pipeline

```text
  Browser file  --PUT-->  Mux Direct Upload URL
                              |
                              v
                     Mux transcode (per-title
                     encoding -> HLS ladder)
                              |
                asset.ready webhook -> save playbackId
                              |
                              v
   <MuxPlayer playbackId>  <---  adaptive HLS  ---  Mux CDN
```

Listen for the `video.asset.ready` webhook to persist the `playbackId`
before showing the player.

------------------------------------------------------------------------

## STEP 6 : Signed Playback (Optional)

For private video set `playback_policy: ["signed"]` and mint a short-lived
JWT server-side, then pass `tokens={{ playback: jwt }}` to `<MuxPlayer>`.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Keep MUX_TOKEN_SECRET server-side; never ship it to the client
✓ Use Direct Uploads so bytes go browser -> Mux, not through you
✓ Persist playbackId only after the asset.ready webhook fires
✓ Use signed playback policy + JWT for gated content
✓ Set streamType correctly (on-demand vs live) for buffering
✓ Pass metadata for useful Mux Data analytics segmentation
✓ Verify webhook signatures with your webhook signing secret
```
