# Quickstart

Add a working blog to an existing Next.js app in under 10 minutes. You'll install the `atom-nextjs` package, create a project on the Atom dashboard, and add two files to your app.

## Prerequisites

- A Next.js 13 or 14 project using the App Router with TailwindCSS configured
- Node.js 18 or later

## 1. Install the packages

```bash
npm install atom-nextjs@latest @tailwindcss/typography
```

## 2. Add atom-nextjs to your Tailwind config

Open `tailwind.config.ts` and add two things: the SDK's source files to `content` (so its class names survive the production build), and the typography plugin:

```ts
// tailwind.config.ts
const config = {
  content: [
    // ... your existing paths ...
    "./node_modules/atom-nextjs/src/components/*.{ts,tsx}",
  ],
  plugins: [
    // ... your existing plugins ...
    require("@tailwindcss/typography"),
  ],
};

export default config;
```

## 3. Sign up for an Atom account

Go to [atomcms.vercel.app/signup](https://atomcms.vercel.app/signup) and create a free account.

## 4. Create a project and write a post

In the dashboard, click **New Project**, give it a name, and hit **Create**. Open the project, then click **New Post** to write your first post. Fill in the title, author, teaser, and body, then save.

## 5. Copy your project key

Inside your project, find the `project_key`  -  it's displayed in the project settings panel. It starts with `atom-`. Copy it.

## 6. Add the key to your environment

Create (or open) `.env.local` in your Next.js project root:

```bash
# .env.local
ATOM_PROJECT_KEY=atom-your-key-here
```

Never commit this file. The key is your Bearer token for the Atom API and it authenticates all requests your blog makes.

## 7. Create the blog index page

Create the file `app/blog/page.tsx`:

```tsx
// app/blog/page.tsx
import { AtomLoadingSkeleton, AtomPage } from "atom-nextjs";
import { cookies } from "next/headers";
import { Suspense } from "react";

export const metadata = { title: "Blog" };

export default function Blog() {
  const _cookies = cookies(); // opts out of caching  -  keeps posts fresh

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

## 8. Create the individual post page

Create the file `app/blog/[id]/page.tsx`:

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from "atom-nextjs";
import { cookies } from "next/headers";
import { Suspense } from "react";

export type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) =>
  generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);

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

## Expected result

Start your dev server with `npm run dev` and open [http://localhost:3000/blog](http://localhost:3000/blog). You should see your post card. Click it to verify the full post renders at `/blog/<post-id>` with the title, author, date, and body.

If the page loads but shows no posts, double-check that `ATOM_PROJECT_KEY` in `.env.local` matches the key shown in your Atom project exactly, including the `atom-` prefix.

## Next steps

- **More styling options**  -  wrap the components in your own layout container, or pass custom `remarkPlugins` and `rehypePlugins` to `Atom` for extended markdown support. The [SDK Guide](./sdk-guide.md) covers all component props and configuration.
- **SEO**  -  add a sitemap using `generateSitemap` so search engines can index your posts. See [SDK Guide  -  Generate a sitemap](./sdk-guide.md#7-generate-a-sitemap).
- **Write more posts**  -  go back to [atomcms.vercel.app](https://atomcms.vercel.app) and publish more posts. They'll appear on your blog index the next time the page is requested.
