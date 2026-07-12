# DISQUS SETUP [ COMMENTS ]
------------------------------------------------------------------------

Disqus is a hosted comments platform with moderation, spam filtering,
and social login out of the box. Choose it when you want a managed
service and don't require readers to hold GitHub accounts. This guide
embeds Disqus in a Next.js App Router blog.

------------------------------------------------------------------------

## STEP 1 : Register a site

1. Sign up at https://disqus.com and choose "I want to install Disqus
   on my site".
2. Pick a `shortname` (your unique site identifier).
3. Select the "Universal Code" / manual install platform.

------------------------------------------------------------------------

## STEP 2 : Store the shortname

```bash
# .env.local  (public: it appears in the embed URL)
NEXT_PUBLIC_DISQUS_SHORTNAME="your-site-shortname"
```

The shortname is public. Windows: edit `.env.local` in VS Code (UTF-8),
never Notepad, to avoid a BOM.

------------------------------------------------------------------------

## STEP 3 : Install the React component

```bash
pnpm add disqus-react
```

------------------------------------------------------------------------

## STEP 4 : Build the client component

Pass a stable identifier + canonical URL so threads never fragment.

```tsx
// components/comments.tsx
"use client";

import { DiscussionEmbed } from "disqus-react";

type Props = { id: string; title: string; url: string };

export function Comments({ id, title, url }: Props) {
  return (
    <DiscussionEmbed
      shortname={process.env.NEXT_PUBLIC_DISQUS_SHORTNAME!}
      config={{
        url, // absolute canonical URL, e.g. https://site.com/blog/x
        identifier: id, // stable per-post key, never the mutable title
        title,
        language: "en",
      }}
    />
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Render on a post

```tsx
// app/blog/[slug]/page.tsx
import { Comments } from "@/components/comments";

export default function Post({ params }: { params: { slug: string } }) {
  const url = `https://yourdomain.com/blog/${params.slug}`;
  return (
    <article>
      {/* ...post content... */}
      <Comments id={params.slug} title="Hello World" url={url} />
    </article>
  );
}
```

------------------------------------------------------------------------

## STEP 6 : Identifier flow

```text
/blog/hello-world
     |
 <Comments id="hello-world" url=".../hello-world">
     |
 DiscussionEmbed -----> disqus.com/embed.js (shortname)
     |                        |
 stable identifier ---------> correct thread (survives URL changes)
 hosted moderation + spam filtering on Disqus dashboard
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Always set a stable identifier so threads survive URL/title edits
✓ Pass an absolute canonical url to prevent duplicate threads
✓ The shortname is public; there are no server secrets to guard
✓ Render as a client component; the embed needs the browser
✓ Enable moderation + spam filtering in the Disqus dashboard
✓ Consider a lazy/on-scroll mount, since the embed is heavy
✓ Pick Disqus for managed moderation and non-GitHub audiences
```
