# Architecture

Atom serves two distinct audiences from a single codebase: developers who want a hosted CMS dashboard to write and manage blog posts, and developers who want to render those posts inside their own Next.js applications. Understanding this dual-purpose design is the key to making sense of the file structure.

## Two products, one repository

**The CMS dashboard** lives at `https://atomcms.vercel.app`. You sign up, create Projects, write Posts using the markdown editor, and copy your `project_key`. Everything here runs on Next.js App Router with session-based authentication  -  the dashboard is a traditional server-rendered web app.

**The `atom-nextjs` SDK** is an NPM package (`packages/atom-nextjs/`) that your blog's visitors interact with, indirectly. You install it into your own Next.js site, drop in two components, and it fetches your published posts from the Atom public API using your `project_key`. Visitors never touch the CMS dashboard.

The two halves share nothing at runtime. The SDK calls the live API over HTTP; it never imports from the main app.

## Directory layout

```
app/                        # Next.js App Router
├── api/                    # REST API route handlers
│   ├── auth/               # signup, signin, signout, delete, user CRUD
│   ├── posts/              # create, delete, update, get
│   └── projects/           # create, delete, get, get/client
├── app/                    # Protected dashboard (/app/*)
│   ├── layout.tsx          # Wraps everything with ProtectedRoute
│   ├── page.tsx            # Projects list
│   ├── projects/[id]/      # Per-project post editor
│   └── settings/           # User settings and billing
├── blog/                   # Example blog powered by atom-nextjs
├── signin/ & signup/       # Public auth pages
└── layout.tsx              # Root layout

lib/
├── client/                 # Axios helpers called from browser/React components
├── server/                 # Server-only: Lucia, Mongoose, Argon2, Redis, utils
├── contants.tsx            # Plan definitions, limits, nav options, env config
└── types.ts                # All shared TypeScript types

packages/
└── atom-nextjs/            # Published NPM package
    └── src/
        ├── components/     # Atom, AtomPage, AtomPostCard, AtomBody, skeletons
        └── lib/client/     # getPost, getProject, generatePostMetadata, generateSitemap
```

The `app/app/` directory (note the double `app`) is the authenticated dashboard. `app/api/` is the REST API. Both sit inside the same Next.js project, which is why the naming is a bit unusual.

## Data flow: CMS dashboard

When a logged-in user creates a post, the request travels through four layers:

```
React component
  → lib/client/* (Axios, runs in the browser)
    → /api/posts/create (Next.js route handler, runs on the server)
      → Mongoose → MongoDB
```

The `lib/client/` helpers are thin wrappers around Axios that handle the `baseAPIRoute` and surface errors as thrown exceptions. They call the same `/api/*` routes that any external HTTP client could call.

Route handlers authenticate every mutating request by calling `validateRequest()`, which reads the Lucia session cookie and returns the current user or `null`.

## Data flow: SDK consumers

When a visitor hits your blog (which uses `atom-nextjs`), the data path is completely different:

```
AtomPage / Atom (React Server Component in consumer's app)
  → getProject / getPost (fetch, runs server-side)
    → https://cmsatom.netlify.app/api/projects/get/single/client
    → https://cmsatom.netlify.app/api/posts/get/single
      → MongoDB (read-only, authenticated by project_key Bearer token)
```

Notice that the SDK components are React Server Components. This keeps the `project_key` out of the browser. Your key is only ever sent in a server-to-server HTTP request.

The `/client` project endpoint (`/api/projects/get/single/client`) deliberately strips the `project_key` field from its response before returning it. This means your key is never leaked to visitors even if they intercept the API response.

## The Project and Post data model

A **Project** is a named blog container owned by one user. It holds an array of **Posts** as embedded subdocuments. The `project_key` on the Project is what SDK consumers use to authenticate.

```
UserDocument (one per account)
└── projects[]  (lightweight refs: id, title, creator)

Project (one collection document per project)
├── project_key     ← the secret your SDK uses
├── creator_uid
└── posts[]         ← embedded Post subdocuments
    ├── id, title, author, teaser
    ├── body (markdown)
    ├── image (URL or null)
    └── keywords[]
```

Posts are embedded inside the Project document rather than stored in a separate collection. This means fetching a project gives you all of its posts in a single query, which keeps the public API fast. The tradeoff is that very large post bodies increase the Project document size.

The `UserDocument` stores a lightweight `projects[]` array (just IDs, titles, and creator info) separately from the full `Project` documents. This lets the dashboard list all your projects cheaply without loading every post.

## Authentication boundary

All `/app/*` dashboard routes are wrapped by `ProtectedRoute`, a server component that calls `validateRequest()` on every render. If there's no valid session, it redirects to `/signin` before any page content is produced.

The `validateRequest` function is wrapped in React's `cache()`, so within a single server render it's called at most once even if multiple components call it.

API routes apply rate limiting at the middleware layer before the route handler runs. Every `/api/*` request is counted against the caller's IP using a 30-requests-per-minute sliding window backed by Upstash Redis. The rate limiter runs regardless of whether the route requires authentication.

## Caching behaviour

`next.config.mjs` sets `Cache-Control: no-store` on `/`, `/app/*`, and `/api/*`. This opts those routes out of Next.js's default full-route cache, ensuring the dashboard always shows fresh data and API responses are never served stale.

The `atom-nextjs` SDK pages use `cookies()` from `next/headers` to opt out of the Next.js data cache. If you want to enable caching on your blog (accepting that posts may be slightly stale), you can remove the `cookies()` call from your blog page components.
