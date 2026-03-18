# SDK Guide

The `atom-nextjs` package gives you a set of React Server Components and utility functions for rendering an Atom-managed blog inside your own Next.js app. This guide walks through every step from installation to sitemap generation.

## Prerequisites

- A Next.js 13 or 14 project using the App Router
- TailwindCSS already set up in your project
- An Atom account with at least one project  -  you'll need the `project_key`

## 1. Install the packages

```bash
npm install atom-nextjs@latest @tailwindcss/typography
```

`@tailwindcss/typography` is required because `Atom` (the single-post component) renders its content inside an `article.prose` element that depends on the typography plugin for styling.

## 2. Configure TailwindCSS

Open your `tailwind.config.ts` (or `.js`) and make two changes: add the SDK's component files to the `content` array so Tailwind can see the class names used inside the package, and register the typography plugin.

```ts
// tailwind.config.ts
const config = {
  content: [
    // ... your existing paths
    "./node_modules/atom-nextjs/src/components/*.{ts,tsx}",
  ],
  plugins: [
    // ... your existing plugins
    require("@tailwindcss/typography"),
  ],
};

export default config;
```

Without the `content` entry, Tailwind will strip the SDK's class names during the production build and your post pages will be unstyled.

## 3. Store your project key

Add your `project_key` to your environment variables. Never commit this value to source control  -  it's the Bearer token that authenticates your app to the Atom API.

```bash
# .env.local
ATOM_PROJECT_KEY=atom-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

The SDK components are React Server Components, so the key is only ever read on the server. It won't appear in your client bundle.

## 4. Create the blog index page

Create `app/blog/page.tsx`. The `AtomPage` component fetches your project from the Atom API and renders a card grid of all published posts.

```tsx
// app/blog/page.tsx
import { AtomLoadingSkeleton, AtomPage } from "atom-nextjs";
import { cookies } from "next/headers";
import { Suspense } from "react";
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "Blog",
};

export default function Blog() {
  // Calling cookies() opts this page out of Next.js's full-route cache,
  // so visitors always see the latest posts.
  const _cookies = cookies();

  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage
        baseRoute="/blog"
        projectKey={process.env.ATOM_PROJECT_KEY!}
      />
    </Suspense>
  );
}
```

`AtomPage` takes two required props:

- `projectKey`  -  your project's secret key
- `baseRoute`  -  the URL prefix for individual posts (e.g. `"/blog"` means each post links to `/blog/<post-id>`)

The optional `title` prop (default `true`) controls whether the project title is rendered as an `<h1>` above the post cards. Set it to `false` if you want to supply your own heading.

`AtomLoadingSkeleton` shows four placeholder cards while the server component streams in. Wrapping `AtomPage` in `Suspense` is optional but strongly recommended for perceived performance.

## 5. Create the individual post page

Create `app/blog/[id]/page.tsx`. The `Atom` component fetches and renders a single post including its full markdown body, author, date, and optional cover image.

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from "atom-nextjs";
import { cookies } from "next/headers";
import { Suspense } from "react";

export type BlogParams = { params: { id: string } };

// Generates <title>, <meta name="description">, keywords, and author tags
// from the post's own data.
export const generateMetadata = async ({ params }: BlogParams) => {
  return await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );
};

export default function BlogPage({ params }: BlogParams) {
  const _cookies = cookies();

  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom
        projectKey={process.env.ATOM_PROJECT_KEY!}
        postId={params.id}
      />
    </Suspense>
  );
}
```

`Atom` takes two required props:

- `projectKey`  -  your project key
- `postId`  -  the post UUID from the URL (this is the `id` field on the `Post` type, not a MongoDB `_id`)

It also accepts optional `remarkPlugins` and `rehypePlugins` arrays if you want to extend MDX rendering. The defaults are `remark-gfm` and `rehype-sanitize`.

`AtomArticleSkeleton` shows a full-width image placeholder and text lines while the post loads.

## 6. Generate post metadata

`generatePostMetadata` calls the Atom API to fetch the post, then returns a Next.js `Metadata` object with:

- `title`  -  the post's title
- `description`  -  the post's teaser
- `keywords`  -  the post's keywords array
- `authors.name`  -  the post's author

If the post can't be found, it returns `{ title: "Couldn't find post.", authors: { name: "Atom" } }` rather than throwing, so your page still renders gracefully.

## 7. Generate a sitemap

Create `app/sitemap.ts` to expose your posts to search engine crawlers. The `generateSitemap` utility fetches your project's post list and returns an array of sitemap entries in Next.js's `MetadataRoute.Sitemap` format.

```ts
// app/sitemap.ts
import { MetadataRoute } from "next";
import { generateSitemap } from "atom-nextjs";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const routes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    "https://yoursite.com/blog"   // the full URL of your blog index
  );

  return [
    {
      url: "https://yoursite.com",
      lastModified: new Date(),
      priority: 0.7,
    },
    ...routes,
  ];
}
```

`generateSitemap` returns one entry per post (with `priority: 0.5` and the post's `updatedAt` as `lastModified`) plus one entry for the blog index itself (with `priority: 0.6`). You can merge these with any other routes in your sitemap.

## Caching behaviour

By default, calling `cookies()` in a page opts it out of Next.js's data cache, ensuring every request fetches fresh posts from the Atom API. This is the right default for a blog with live updates.

If you want to cache the blog index (accepting that newly published posts may not appear immediately), remove the `cookies()` call:

```tsx
export default function Blog() {
  // No cookies() call  -  Next.js will cache this page normally
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
    </Suspense>
  );
}
```

## Dark mode

The `atom-nextjs` SDK currently does not support dark mode. The prose and card styles are light-mode only. If your site uses `dark:` Tailwind variants, post pages will not adapt to dark mode automatically.

## Markdown and MDX

Post bodies are compiled with [`next-mdx-remote`](https://github.com/hashicorp/next-mdx-remote). The compiler runs server-side inside `AtomBody`. GitHub-Flavored Markdown (tables, strikethrough, task lists) is enabled by default via `remark-gfm`. HTML in the markdown is sanitised via `rehype-sanitize`.

If you need additional remark or rehype plugins (for math, syntax highlighting, etc.), pass them to `Atom`:

```tsx
import remarkMath from "remark-math";
import rehypeKatex from "rehype-katex";

<Atom
  projectKey={process.env.ATOM_PROJECT_KEY!}
  postId={params.id}
  remarkPlugins={[remarkMath]}
  rehypePlugins={[rehypeKatex]}
/>
```

Custom plugins are appended after the defaults, so GFM and sanitisation remain active.
