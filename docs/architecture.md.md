# Architecture

This page explains how Atom is structured, how its pieces communicate, and why the system is designed the way it is.

## The two-part system

Atom is split into two independently useful pieces:

1. **The Dashboard App** is a full-stack Next.js 14 application. It serves the marketing site, authentication flows, a protected dashboard for managing projects and posts, and a public REST API. This is the code in the root of the repository.

2. **The SDK Package** (`atom-nextjs`) is a published npm package that consumer Next.js apps install to fetch and render blog content. It lives in `packages/atom-nextjs/` and is built with `tsdx`. It exports React server components and helper functions.

The dashboard is where content gets created. The SDK is how that content gets displayed. The REST API is the bridge between them.

## Directory structure

Here's how the codebase is organized:

```
atom/
├── app/                          # Next.js App Router
│   ├── api/                      # REST API (14 endpoints)
│   │   ├── auth/                 # signup, signin, signout, delete, user/get, user/update
│   │   ├── posts/                # create, delete, update, get/single
│   │   └── projects/             # create, delete, get/single, get/single/client
│   ├── app/                      # Protected dashboard pages
│   │   ├── projects/[id]/        # Single project view
│   │   ├── settings/             # User settings
│   │   └── settings/billing/     # Billing page
│   ├── blog/                     # Public blog (renders using the SDK itself)
│   ├── signin/                   # Sign-in page
│   ├── signup/                   # Sign-up page
│   └── pricing/                  # Pricing page
├── components/                   # React components (forms, modals, nav, tables, UI)
├── lib/                          # Shared logic
│   ├── types.ts                  # Core TypeScript types
│   ├── contants.tsx              # Plans, nav config, limits, API base URL
│   ├── client/                   # Client-side API wrappers (axios)
│   └── server/                   # Server-side utilities
│       ├── encoding/             # Argon2 password hashing
│       ├── lucia/                # Auth initialization and session validation
│       ├── mongo/                # Mongoose models and connection
│       ├── redis/                # Upstash rate limiter
│       └── utils/                # Project key generation
├── packages/atom-nextjs/         # The SDK package
│   └── src/
│       ├── index.tsx             # Package entry point (all exports)
│       ├── components/           # Atom, AtomPage, AtomBody, AtomPostCard, skeletons
│       └── lib/                  # Fetch functions, metadata, sitemap, types
└── middleware.ts                 # Rate limiting middleware for /api routes
```

## How data flows

The simplest way to understand Atom is to follow a blog post from creation to rendering.

### Step 1: User creates content in the dashboard

A logged-in user visits `/app`, creates a project, and writes posts inside it. The dashboard's React components call client-side wrapper functions in `lib/client/`, which make axios requests to the API routes in `app/api/`.

When a project is created, the server generates a unique `project_key` (a `atom-` prefixed base64 string from 32 random bytes). This key is the credential that external apps use to read the project's content.

### Step 2: Content is stored in MongoDB

All data lives in four MongoDB collections:

| Collection    | What it stores                                             |
| ------------- | ---------------------------------------------------------- |
| `credentials` | Email + Argon2 password hash per user                      |
| `documents`   | User profiles (name, email, plan, project summaries)       |
| `projects`    | Full project documents with posts embedded as subdocuments |
| `sessions`    | Lucia auth sessions                                        |

Posts are not stored in their own collection. They're embedded as an array inside the project document. This means fetching a project returns all its posts in one query.

### Step 3: External apps fetch via the public API

A consumer Next.js app installs `atom-nextjs` and passes the `project_key` as a Bearer token. The SDK calls two public API endpoints:

- **`GET /api/projects/get/single/client`** returns the project with a sanitized list of posts (no body content, no sensitive fields like `creator_uid` or `project_key`).
- **`GET /api/posts/get/single?post_id=X`** returns a single post with full body content for rendering.

This two-step approach means the post listing page loads quickly (no body content), and the full MDX body is only fetched when a reader opens a specific post.

### Step 4: The SDK renders the content

The SDK's React server components handle the rendering:

- `AtomPage` fetches the project and displays a grid of post cards.
- `Atom` fetches a single post and renders its MDX body using `next-mdx-remote`.
- `AtomBody` handles the MDX-to-HTML conversion with plugins like `remark-gfm` and `rehype-sanitize`.

Because these are server components, the blog content is rendered on the server and delivered as HTML. No client-side fetching or loading spinners for the content itself.

## Authentication model

Atom uses Lucia Auth with a MongoDB adapter. Here's the flow:

1. On signup, the server hashes the password with Argon2 (using a salt from the `HASH_SALT` environment variable), creates credential and document records in a transaction, and sets a session cookie.
2. On each request to a protected route, the `ProtectedRoute` server component calls `validateRequest()`. This function (wrapped in React's `cache()` so it only runs once per request) reads the session cookie, validates it with Lucia, and refreshes it if it's close to expiry.
3. If the session is invalid, the user is redirected to `/signin`.

API endpoints that manage content (creating projects, editing posts) validate the session cookie directly. Public API endpoints (fetching posts for rendering) use Bearer token authentication with the `project_key` instead.

## Rate limiting

All `/api` routes pass through the middleware in `middleware.ts`, which applies an Upstash Redis sliding window rate limiter: 30 requests per minute per IP address. If the limit is exceeded, the middleware returns a JSON error response without forwarding the request to the route handler.

The rate limiter is configured in `lib/server/redis/init.ts`:

```typescript
export const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(30, "1 m"),
});
```

## Key design decisions

**Posts are embedded in projects, not stored separately.** This simplifies queries (one fetch gets everything) but means that projects with thousands of posts could hit document size limits. The plan system mitigates this by capping post counts (100 for Single, 1,000 for Startup, 2,500 for Business).

**The project key is both an identifier and a credential.** The `project_key` is used as a Bearer token in the Authorization header. This keeps the SDK integration simple (one string to configure) but means that anyone with the key can read all posts in the project. Since blog content is typically public anyway, this tradeoff makes sense.

**The SDK uses server components, not client-side fetching.** This means blog content is rendered at build time or request time on the server. It's better for SEO and initial load performance, but it means the SDK only works in Next.js apps that support React Server Components.

**User documents are denormalized.** The `documents` collection contains a `projects` array with summaries (title, ID, timestamps), while the full project data lives in the `projects` collection. This means the dashboard can show a project list without joining collections, but project renames need to update both places.

**Watermarks on the free plan.** When a post is fetched through the public API for a user on the Single (free) plan, the server appends a Markdown watermark to the body: "This post was created using Atom." Paid plans remove it.

## Testing 123
