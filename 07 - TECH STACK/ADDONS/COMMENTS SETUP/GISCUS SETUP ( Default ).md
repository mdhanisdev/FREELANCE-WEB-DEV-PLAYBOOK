# GISCUS SETUP [ COMMENTS ]
------------------------------------------------------------------------

giscus is the default comments system for this stack: it stores every
comment thread in your repo's GitHub Discussions, so there is no
database, no vendor lock-in, and no ads. Readers sign in with GitHub.
This guide wires giscus into a Next.js App Router blog.

------------------------------------------------------------------------

## STEP 1 : Prepare the GitHub repo

1. The repo must be public.
2. Settings -> General -> Features: enable Discussions.
3. Install the giscus app: https://github.com/apps/giscus and grant it
   access to the repo.

------------------------------------------------------------------------

## STEP 2 : Generate configuration

Open https://giscus.app, enter `owner/repo`, and pick a Discussion
category (e.g. "Announcements"). The page returns the four IDs you need.

```bash
# .env.local  (all public: they ship in the client bundle)
NEXT_PUBLIC_GISCUS_REPO="owner/repo"
NEXT_PUBLIC_GISCUS_REPO_ID="R_kgDO..."
NEXT_PUBLIC_GISCUS_CATEGORY="Announcements"
NEXT_PUBLIC_GISCUS_CATEGORY_ID="DIC_kwDO..."
```

Windows: save `.env.local` as UTF-8 in VS Code, not Notepad (no BOM).

------------------------------------------------------------------------

## STEP 3 : Install the React wrapper

```bash
pnpm add @giscus/react
```

------------------------------------------------------------------------

## STEP 4 : Build the client component

```tsx
// components/comments.tsx
"use client";

import Giscus from "@giscus/react";

export function Comments() {
  return (
    <Giscus
      repo={process.env.NEXT_PUBLIC_GISCUS_REPO as `${string}/${string}`}
      repoId={process.env.NEXT_PUBLIC_GISCUS_REPO_ID!}
      category={process.env.NEXT_PUBLIC_GISCUS_CATEGORY!}
      categoryId={process.env.NEXT_PUBLIC_GISCUS_CATEGORY_ID!}
      mapping="pathname"
      reactionsEnabled="1"
      emitMetadata="0"
      inputPosition="top"
      theme="preferred_color_scheme"
      lang="en"
      loading="lazy"
    />
  );
}
```

------------------------------------------------------------------------

## STEP 5 : Drop it into a post

```tsx
// app/blog/[slug]/page.tsx
import { Comments } from "@/components/comments";

export default function Post() {
  return (
    <article>
      {/* ...post content... */}
      <Comments />
    </article>
  );
}
```

`mapping="pathname"` ties one Discussion to each URL automatically.

------------------------------------------------------------------------

## STEP 6 : How it maps

```text
/blog/hello-world  --------- pathname mapping --------->  GitHub
     |                                                    Discussion
 <Comments/> loads giscus iframe                          "hello-world"
 reader signs in with GitHub  ------------------------->  posts comment
 comment stored in repo Discussions (you own the data)
```

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Use mapping="pathname" so each page maps to one stable Discussion
✓ Keep all four IDs public; there are no secrets in giscus
✓ Set loading="lazy" so the iframe never blocks first paint
✓ Use theme="preferred_color_scheme" to match light/dark automatically
✓ Pick a dedicated Discussion category to keep threads organized
✓ Render <Comments/> as a client component; it needs the browser
✓ Keep giscus as the default: zero-cost, no database, you own the data
```
