# SDK Guide

This guide walks you through integrating the `atom-nextjs` SDK into a consumer Next.js app. By the end, you'll have a blog listing page, individual post pages, proper loading states, SEO metadata, and a sitemap.

## Prerequisites

- A Next.js 14+ app using the App Router
- TailwindCSS configured in your project (the SDK components use Tailwind classes)
- An Atom project with at least one published post
- Your project's `project_key` (found in the project settings on the Atom dashboard)

## Installation

```bash
npm i atom-nextjs@latest
```

The SDK has a few peer/transitive dependencies that it bundles: `next-mdx-remote` for MDX rendering, `react-loading-skeleton` for loading states, `remark-gfm` for GitHub Flavored Markdown, and `rehype-sanitize` for HTML sanitization.

## Store your project key

Add your project key to your `.env` file:

```env
ATOM_PROJECT_KEY=atom-your-project-key-here
```

Use a server-side environment variable (no `NEXT_PUBLIC_` prefix). The SDK components are server components, so they can read server-only env vars directly. This keeps your project key out of client-side bundles.

## TailwindCSS configuration

The SDK components use Tailwind's `prose` classes from the `@tailwindcss/typography` plugin. If you haven't installed it yet:

```bash
npm i @tailwindcss/typography
```

Add it to your `tailwind.config.ts`:

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx}",
    // Include atom-nextjs so Tailwind picks up its classes
    "./node_modules/atom-nextjs/**/*.{js,ts,jsx,tsx}",
  ],
  plugins: [
    require("@tailwindcss/typography"),
  ],
};

export default config;
```

The important part is adding `./node_modules/atom-nextjs/**/*.{js,ts,jsx,tsx}` to the `content` array. Without this, Tailwind will purge the CSS classes used by SDK components and they'll render unstyled.

## Blog listing page

Create a blog page that displays all posts in your project as a grid of cards.

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from "atom-nextjs";
import { Suspense } from "react";

export default function Blog() {
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

`AtomPage` is an async server component. It calls `getProject()` internally to fetch the sanitized project data from the Atom API, then renders an `AtomPostCard` for each post.

The props:

- **`projectKey`** is your project key, passed as a Bearer token to the API.
- **`baseRoute`** sets the URL prefix for post links. Each card links to `{baseRoute}/{post.id}`. So with `baseRoute="/blog"`, a post with ID `abc123` links to `/blog/abc123`.
- **`title`** (optional, defaults to `true`) controls whether the project title is displayed as an `<h1>` above the post grid.

The `Suspense` boundary with `AtomLoadingSkeleton` shows a grid of placeholder cards while the API request is in flight. This matters because `AtomPage` is an async component that fetches data during server rendering.

## Individual post page

Create a dynamic route for rendering single posts with their full MDX body.

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from "atom-nextjs";
import { Suspense } from "react";

type BlogPostParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogPostParams) => {
  return await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );
};

export default function BlogPost({ params }: BlogPostParams) {
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

`Atom` is the main post rendering component. It fetches the full post (including the MDX body) via `getPost()`, then renders:

1. The post title as a centered `<h1>`
2. The cover image (if one exists) as a full-width banner
3. The author name and publish date
4. The MDX body, rendered by `AtomBody` using `next-mdx-remote`

The component wraps everything in an `<article>` with Tailwind's `prose lg:prose-xl` classes for typographic styling.

The `Atom` component also accepts optional `remarkPlugins` and `rehypePlugins` props if you want to extend the MDX processing pipeline beyond the defaults (`remark-gfm` and `rehype-sanitize`).

## Metadata generation

The `generateMetadata` export in the post page uses `generatePostMetadata()` to fetch the post and return a Next.js `Metadata` object. It sets:

- `title` from the post title
- `description` from the post teaser
- `keywords` from the post keywords array
- `authors` from the post author name

If the post can't be found, it returns a fallback with `title: "Couldn't find post."` and `authors: { name: "Atom" }`.

This function makes an API call to fetch the post, so it runs on the server during rendering. Next.js deduplicates this with the `Atom` component's fetch if they happen in the same request.

## Sitemap generation

Create a `sitemap.ts` file in your `app/` directory to generate a sitemap that includes all your blog posts:

```typescript
// app/sitemap.ts
import { MetadataRoute } from "next";
import { generateSitemap } from "atom-nextjs";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const routes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    "https://yourdomain.com/blog"
  );

  return [
    {
      url: "https://yourdomain.com",
      lastModified: new Date(),
      priority: 0.7,
    },
    ...routes,
  ];
}
```

`generateSitemap()` fetches your project and returns an array of sitemap entries. Each post gets a URL of `{blogRoute}/{post.id}`, a `lastModified` timestamp from the post's `updatedAt` field, and a priority of `0.5`. The blog index page itself is included with a priority of `0.6`.

You can spread these entries alongside your other site routes, as shown above.

## Loading states

The SDK provides two skeleton components for loading states:

**`AtomLoadingSkeleton`** renders a grid of four placeholder cards. Use it as the `Suspense` fallback for `AtomPage`:

```tsx
<Suspense fallback={<AtomLoadingSkeleton />}>
  <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
</Suspense>
```

**`AtomArticleSkeleton`** renders a single-article placeholder with a large image area and text lines. Use it as the `Suspense` fallback for `Atom`:

```tsx
<Suspense fallback={<AtomArticleSkeleton />}>
  <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
</Suspense>
```

Both skeletons use `react-loading-skeleton` internally. They match the layout of the corresponding content components so the page doesn't jump when the real content loads.

## Using the fetch functions directly

If you need more control over rendering, you can use `getProject()` and `getPost()` directly instead of the pre-built components.

`getProject()` fetches the sanitized project (post listings without body content):

```typescript
import { getProject } from "atom-nextjs";

const result = await getProject(process.env.ATOM_PROJECT_KEY!);

if (result.success) {
  const project = result.response;
  // project.title, project.posts, project.id, etc.
}
```

`getPost()` fetches a single post with full body content:

```typescript
import { getPost } from "atom-nextjs";

const result = await getPost(process.env.ATOM_PROJECT_KEY!, "post-id-here");

if (result.success) {
  const post = result.response;
  // post.title, post.body, post.author, etc.
}
```

Both functions return an `ApiResponse<T>` object with `success`, `message`, and `response` fields. Always check `success` before accessing `response`.

## Rendering MDX independently

If you're fetching posts with `getPost()` and want to render the MDX body yourself, use `AtomBody`:

```tsx
import { AtomBody } from "atom-nextjs";

// Inside an async server component:
<AtomBody
  body={post.body}
  className="space-y-6"
  remarkPlugins={[/* your plugins */]}
  rehypePlugins={[/* your plugins */]}
/>
```

`AtomBody` is an async server component that compiles MDX using `next-mdx-remote/rsc`. By default it includes `remark-gfm` (tables, strikethrough, autolinks) and `rehype-sanitize` (strips unsafe HTML). Any plugins you pass are appended to these defaults, not replacements.

## How the SDK communicates with Atom

Understanding the API calls helps with debugging and caching decisions:

1. `AtomPage` (and `getProject()`) calls **`GET /api/projects/get/single/client`** with the project key as a Bearer token. This returns the project with sanitized posts (no body content).

2. `Atom` (and `getPost()`) calls **`GET /api/posts/get/single?post_id=X`** with the project key as a Bearer token. This returns the full post including the MDX body.

Both calls go to the production Atom API at `https://cmsatom.netlify.app/api`. The SDK's base URL is hardcoded to this production endpoint. This means your consumer app always talks to the hosted Atom instance, not a local development server.

Since these are server-side `fetch()` calls in server components, Next.js will cache them according to its default caching behavior. If you need fresh data on every request, you can opt out of caching by calling `cookies()` in the page (which forces dynamic rendering) or by configuring the route segment.
