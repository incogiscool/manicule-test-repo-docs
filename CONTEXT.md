Now I have a thorough understanding of the entire codebase. Let me compile the CONTEXT.md:

---

# CONTEXT.md

# Atom CMS

## Overview

Atom is a headless Content Management System (CMS) built for Next.js developers. It provides a full-stack web application (the Atom platform) where users register, create "projects" (blog/content collections), and write/publish posts using a Markdown editor. It ships alongside a companion npm package (`atom-nextjs`) that consumer Next.js apps install to render blog content via pre-built React server components. The core stack is **Next.js 14 (App Router)**, **TypeScript**, **MongoDB (Mongoose)**, **Lucia v3** (session-based auth), **Upstash Redis** (rate limiting), **Tailwind CSS**, and **shadcn/ui** component primitives.

---

## File Tree

```
context/test-code/                        ← Root of the Atom CMS application
├── .eslintrc.json
├── .gitignore
├── LICENSE
├── README.md
├── bun.lock
├── components.json                       ← shadcn/ui configuration
├── middleware.ts                         ← Next.js middleware (rate limiting on /api/*)
├── next.config.mjs                       ← Next.js config (Cache-Control headers)
├── package.json
├── package-lock.json
├── postcss.config.js
├── tailwind.config.ts
├── tsconfig.json
│
├── app/                                  ← Next.js App Router pages & API routes
│   ├── favicon.ico
│   ├── globals.css                       ← Tailwind base + CSS custom properties
│   ├── layout.tsx                        ← Root layout (Montserrat font, Toaster)
│   ├── page.tsx                          ← Public marketing home page
│   ├── robots.ts                         ← robots.txt generation
│   ├── sitemap.ts                        ← Dynamic sitemap (uses atom-nextjs SDK)
│   │
│   ├── pricing/
│   │   └── page.tsx                      ← Public pricing page
│   ├── signin/
│   │   └── page.tsx                      ← Sign-in page (LoginForm)
│   ├── signup/
│   │   └── page.tsx                      ← Sign-up page (SignupForm)
│   │
│   ├── blog/
│   │   ├── page.tsx                      ← Public blog listing (AtomPage from SDK)
│   │   └── [id]/
│   │       └── page.tsx                  ← Public blog post page (Atom from SDK)
│   │
│   ├── app/                              ← Authenticated application area
│   │   ├── layout.tsx                    ← Wraps children with ProtectedRoute
│   │   ├── page.tsx                      ← Projects dashboard (/app)
│   │   ├── projects/
│   │   │   └── [id]/
│   │   │       └── page.tsx              ← Single project editor page
│   │   └── settings/
│   │       ├── page.tsx                  ← User settings page
│   │       └── billing/
│   │           └── page.tsx              ← Billing page (Coming soon)
│   │
│   └── api/                              ← REST API route handlers
│       ├── auth/
│       │   ├── signup/route.ts           ← POST /api/auth/signup
│       │   ├── signin/route.ts           ← POST /api/auth/signin
│       │   ├── signout/route.ts          ← POST /api/auth/signout
│       │   ├── delete/route.ts           ← DELETE /api/auth/delete
│       │   └── user/
│       │       ├── get/route.ts          ← GET /api/auth/user/get
│       │       └── update/route.ts       ← PATCH /api/auth/user/update
│       ├── posts/
│       │   ├── create/route.ts           ← POST /api/posts/create?project_id=
│       │   ├── delete/route.ts           ← DELETE /api/posts/delete?project_id=&post_id=
│       │   ├── update/route.ts           ← PATCH /api/posts/update?project_id=&post_id=
│       │   └── get/single/route.ts       ← GET /api/posts/get/single?post_id= (Bearer auth)
│       └── projects/
│           ├── create/route.ts           ← POST /api/projects/create
│           ├── delete/route.ts           ← DELETE /api/projects/delete?project_id=
│           └── get/single/
│               ├── route.ts              ← GET /api/projects/get/single (session or Bearer)
│               └── client/route.ts       ← GET /api/projects/get/single/client (Bearer, public)
│
├── components/                           ← Reusable React components
│   ├── cards/
│   │   └── PricingPlanCard.tsx
│   ├── containers/
│   │   ├── AppContainer.tsx             ← Sidebar + main layout for /app area
│   │   ├── MainContainer.tsx            ← Navbar + footer layout for public pages
│   │   └── ProtectedRoute.tsx           ← Server component: validates session, redirects
│   ├── forms/
│   │   ├── LoginForm.tsx                ← Client form (react-hook-form + zod)
│   │   └── SignupForm.tsx               ← Client form (react-hook-form + zod)
│   ├── misc/
│   │   ├── NpmPackageComponent.tsx      ← Copyable npm install command display
│   │   └── tracing-beam.tsx             ← Framer Motion decorative component
│   ├── modals/
│   │   ├── CreatePostModal.tsx          ← Dialog to create a new post
│   │   └── DeleteUserModal.tsx          ← Dialog to confirm account deletion
│   ├── nav/
│   │   └── Navbar.tsx                   ← Top navigation bar (server component)
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectComponent.tsx     ← Split-pane project editor (posts list + form)
│   │   │   ├── ProjectFormComponent.tsx ← Edit/delete a single post form
│   │   │   └── ProjectPage.tsx          ← Projects dashboard with DataTable
│   │   └── settings/
│   │       └── SettingsForm.tsx         ← Update first/last name form
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx            ← Left sidebar nav in /app area
│   │   └── ProjectComponentSidebar.tsx  ← Post list sidebar inside a project
│   ├── tables/
│   │   └── UserDocumentProjects/
│   │       ├── columns.tsx              ← TanStack Table column definitions
│   │       └── table.tsx                ← DataTable wrapper component
│   └── ui/                              ← shadcn/ui primitives
│       ├── alert-dialog.tsx
│       ├── button.tsx
│       ├── carousel.tsx
│       ├── dialog.tsx
│       ├── dropdown-menu.tsx
│       ├── form.tsx
│       ├── input.tsx
│       ├── label.tsx
│       ├── markdown-editor.tsx          ← @uiw/react-md-editor wrapper
│       ├── popover.tsx
│       ├── sticky-scroll-reveal.tsx
│       ├── table.tsx
│       └── textarea.tsx
│
├── lib/
│   ├── contants.tsx                     ← App-wide constants (plans, navOptions, limits, base API URL)
│   ├── types.ts                         ← All TypeScript types/interfaces
│   ├── utils.ts                         ← cn() Tailwind merge utility
│   ├── utils/
│   │   └── validateEmail.ts             ← Email regex validation
│   ├── client/                          ← Client-side API call wrappers (axios)
│   │   ├── auth/
│   │   │   ├── loginUser.ts
│   │   │   ├── signupUser.ts
│   │   │   ├── signoutUser.ts
│   │   │   ├── updateUser.ts
│   │   │   └── deleteUser.ts
│   │   ├── posts/
│   │   │   ├── createPost.ts
│   │   │   ├── deletePost.ts
│   │   │   └── updatePost.ts
│   │   └── projects/
│   │       ├── createProject.ts
│   │       └── deleteProject.ts
│   └── server/                          ← Server-only utilities
│       ├── encoding/
│       │   ├── encodePassword.ts        ← argon2 hash + salt
│       │   └── isPasswordValid.ts       ← argon2 verify
│       ├── functions/
│       │   ├── projects/
│       │   │   └── getProject.ts        ← Server-side axios fetch for a project
│       │   └── user/
│       │       └── fetchUser.ts         ← Server-side axios fetch for current user
│       ├── lucia/
│       │   ├── init.ts                  ← Lucia auth instance + MongoDB adapter
│       │   └── functions/
│       │       └── validate-request.ts  ← Cached session validator (reads cookies)
│       ├── mongo/
│       │   ├── init.ts                  ← Mongoose connection + model refs
│       │   └── types/
│       │       ├── userCredentials.ts   ← Mongoose schema: credentials collection
│       │       ├── userDocuments.ts     ← Mongoose schema: documents collection
│       │       ├── userProjects.ts      ← Mongoose schema: projects collection
│       │       └── userSessions.ts      ← Mongoose schema: sessions collection
│       ├── redis/
│       │   └── init.ts                  ← Upstash Redis + Ratelimit instances
│       └── utils/
│           ├── generateProjectKey.ts    ← crypto.randomBytes → "atom-<base64>" key
│           ├── validateProjectKey.ts    ← Checks project_key in DB
│           └── validateRequestFetchUser.ts ← validateRequest() + UserDocumentsRef lookup
│
├── public/
│   ├── atom-black.svg
│   ├── next.svg
│   └── vercel.svg
│
└── packages/
    └── atom-nextjs/                     ← Publishable npm SDK (atom-nextjs@0.3.1)
        ├── package.json
        ├── tsconfig.json
        ├── README.md
        ├── yarn.lock
        ├── .gitignore
        ├── .github/workflows/
        │   ├── main.yml
        │   └── size.yml
        └── src/
            ├── index.tsx                ← Package public exports
            ├── components/
            │   ├── Atom.tsx             ← Server component: renders a full post
            │   ├── AtomBody.tsx         ← Async MDX body renderer (compileMDX)
            │   ├── AtomPage.tsx         ← Server component: renders post list
            │   ├── AtomPostCard.tsx     ← Individual post card with link
            │   ├── AtomLoadingSkeleton.tsx  ← Skeleton for post listing
            │   └── AtomArticleSkeleton.tsx  ← Skeleton for single article
            └── lib/
                ├── types.ts             ← SDK type definitions
                ├── constants.ts         ← SDK base API URL
                └── client/
                    ├── getPost.ts       ← fetch post by key + ID (Bearer)
                    ├── getProject.ts    ← fetch client-safe project (Bearer)
                    ├── generatePostMetadata.ts ← Next.js Metadata generation
                    └── generateSitemap.ts      ← Next.js Sitemap generation
```

---

## Architecture

The project is a **monorepo** with two parts:

### 1. Main Application (`context/test-code/`)

A Next.js 14 App Router application serving two distinct roles:

**Public-facing marketing/blog site** (`/`, `/blog`, `/pricing`, `/signin`, `/signup`)
- Uses `MainContainer` (Navbar + footer wrapper) for layout.
- The `/blog` routes consume the `atom-nextjs` SDK with the app's own `ATOM_PROJECT_KEY` — the site demos its own product.

**Authenticated CMS dashboard** (`/app`, `/app/projects/[id]`, `/app/settings`)
- Protected by `ProtectedRoute` server component (in `app/app/layout.tsx`) which calls `validateRequest()` and redirects unauthenticated users to `/signin`.
- Uses `AppContainer` (sidebar + main content) for layout.

**API Layer** (`/api/**`)
- All routes are plain Next.js Route Handlers (no separate Express server).
- Rate-limited at 30 requests/minute per IP via Next.js middleware using Upstash Redis sliding window.
- Auth routes use Lucia session cookies; public data routes use Bearer project key.
- All handlers follow a consistent try/catch pattern returning `ApiResponse<T>`.

**Data Flow (typical authenticated request):**
1. Client component calls a `lib/client/**` function (axios).
2. Axios POSTs/GETs the Next.js API route.
3. Middleware checks rate limit for `/api/*`.
4. Route handler calls `connectToDatabase()`, then `validateRequest()` to verify Lucia session.
5. Route handler reads/writes MongoDB via Mongoose model refs.
6. Returns `ApiResponse<T>` JSON.

### 2. `atom-nextjs` SDK (`packages/atom-nextjs/`)

A separate publishable npm package (v0.3.1, bundled with `tsdx`). Consumer Next.js apps install this to render their Atom CMS blog. It exclusively uses **fetch** calls with a `Bearer <project_key>` header to the Atom API (`cmsatom.netlify.app/api`). All SDK components are **Next.js server components** to keep the project key server-side.

---

## Key Files

### Configuration
- **`next.config.mjs`** — Sets `Cache-Control: no-store` on `/`, `/app/**`, and `/api/**` to prevent stale data.
- **`tailwind.config.ts`** — Configures dark mode by class, content paths (including SDK components), shadcn/ui CSS variable color system, accordion animations, and registers `tailwindcss-animate`, `@tailwindcss/typography`, and a custom `addVariablesForColors` plugin.
- **`components.json`** — shadcn/ui configuration; aliases `@/components` and `@/lib/utils`.
- **`tsconfig.json`** — Path alias `@/*` maps to the project root.
- **`lib/contants.tsx`** — Single source of truth for plan definitions, nav options, character limits, the `baseAPIRoute` (switches between prod/dev by `NEXT_PUBLIC_ENV`), and MongoDB URI export.

### Auth & Session
- **`lib/server/lucia/init.ts`** — Initializes Lucia with a MongoDB adapter pointing at the `sessions` and `credentials` collections. Session cookies never expire and are secure in production.
- **`lib/server/lucia/functions/validate-request.ts`** — React `cache()`-wrapped function that reads the session cookie, validates it against Lucia, and refreshes/clears the cookie as needed. Used by every protected route handler and server component.
- **`lib/server/encoding/encodePassword.ts`** / **`isPasswordValid.ts`** — Password is salted with `HASH_SALT` env var then hashed/verified with `argon2`.

### Database
- **`lib/server/mongo/init.ts`** — Mongoose connection and the four model singletons: `UserCredentialsRef`, `UserDocumentsRef`, `ProjectsRef`, `SessionRef`.
- **`lib/server/redis/init.ts`** — Upstash Redis client and a `Ratelimit` instance (sliding window, 30 req/min).

### Types
- **`lib/types.ts`** — All domain TypeScript types: `UserCredentials`, `UserDocument`, `Post`, `Project`, `Session`, `Plan`, `UserDocumentProjects`, `PlanDetailsPlan`, `NavOptionIds`.

### API Routes (see API section for full list)
- **`app/api/auth/signup/route.ts`** — Defines and exports the `ApiResponse<T>` generic type used across all routes.
- **`app/api/projects/get/single/client/route.ts`** — The public-facing endpoint used by the SDK; returns only `ClientProject` (no post bodies, no project keys).

### Frontend Components
- **`components/containers/ProtectedRoute.tsx`** — Server component; checks session and `redirect()`s to `/signin` if unauthenticated.
- **`components/pages/projects/ProjectComponent.tsx`** — The main CMS editor: a two-pane layout with a posts sidebar and a post edit form.
- **`components/pages/projects/ProjectFormComponent.tsx`** — Edit/delete a specific post; includes a `MarkdownEditor` and a Zod-validated form.
- **`components/modals/CreatePostModal.tsx`** — Dialog form with all post fields including Markdown body editor.
- **`components/tables/UserDocumentProjects/`** — TanStack Table implementation for the projects dashboard list.

### SDK (`packages/atom-nextjs/`)
- **`src/index.tsx`** — Barrel file exporting all public SDK symbols.
- **`src/components/Atom.tsx`** — Primary consumer component: fetches post by ID and renders title, image, author, date, and MDX body.
- **`src/components/AtomBody.tsx`** — Async server component using `compileMDX` (next-mdx-remote/rsc) with `remarkGfm` and `rehypeSanitize`.
- **`src/components/AtomPage.tsx`** — Renders a grid of `AtomPostCard` components for the blog listing page.
- **`src/lib/constants.ts`** — Contains the hard-coded base API URL `https://cmsatom.netlify.app/api`.

---

## Data Model

### MongoDB Collections

#### `credentials` — `UserCredentials`
```ts
{
  _id: string;           // UUID (= user ID)
  email: string;         // unique, lowercase, trimmed
  password_hash: string; // argon2 hash
  createdAt: Date;       // auto via timestamps
  updatedAt: Date;       // auto via timestamps
}
```

#### `documents` — `UserDocument`
```ts
{
  _id: string;            // UUID (= user ID, same as credentials._id)
  first_name: string;
  last_name: string;
  email: string;
  plan: "single" | "startup" | "business";
  projects: UserDocumentProjects[];  // denormalised project references
  createdAt: Date;
  updatedAt: Date;
}

UserDocumentProjects = {
  id: string;             // Project._id
  title: string;
  createdAt: Date;
  updatedAt: Date;
  creator: { uid: string; email: string; }
}
```

#### `projects` — `Project`
```ts
{
  _id: string;            // UUID
  title: string;
  posts: Post[];          // embedded array
  project_key: string;   // "atom-<32 random base64 bytes>"
  creator_uid: string;   // references UserDocument._id
  createdAt: Date;
  updatedAt: Date;
}

Post = {
  id: string;            // UUID (not Mongo _id)
  title: string;
  author: string;
  body: string;          // Markdown / MDX content
  teaser: string;        // max 100 chars
  image: string | null;  // URL
  keywords?: string[];
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
}
```

#### `session` — `Session`
```ts
{
  user_id: string;   // references credentials._id
  expires_at: Date;
}
```

### Plan Definitions (from `lib/contants.tsx`)
| Plan | Price | Max Projects | Max Posts | Max Body Length |
|------|-------|-------------|-----------|----------------|
| single | Free | 2 | 100 | 10,000 chars |
| startup | $3.99/mo | 3 | 1,000 | 100,000 chars |
| business | $11.99/mo | 5 | 5,000 | 500,000 chars |

Plans `startup` and `business` are defined but `disabled: true` (coming soon). Posts from `single` plan users get an Atom watermark appended to `body` when fetched via the public API.

---

## API / Routes

All API responses follow this shape (defined in `app/api/auth/signup/route.ts`):
```ts
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
}
```

All `/api/*` routes are rate-limited to **30 requests per minute per IP** (sliding window, Upstash Redis).

### Auth Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/auth/signup` | None | Create account. Body: `{ email, password, first_name, last_name }`. Creates `UserCredentials` + `UserDocument` in a transaction, sets Lucia session cookie. Returns `ApiResponse<UserDocument>`. |
| `POST` | `/api/auth/signin` | None | Sign in. Body: `{ email, password }`. Verifies argon2 hash, creates Lucia session cookie. Returns `ApiResponse<UserDocument>`. |
| `POST` | `/api/auth/signout` | Session Cookie | Invalidates Lucia session, clears cookie. |
| `DELETE` | `/api/auth/delete` | Session Cookie | Permanently deletes user credentials, documents, projects, and sessions in a transaction. Body: `{ password }` for confirmation. |
| `GET` | `/api/auth/user/get` | Session Cookie | Returns current user's `UserDocument`. |
| `PATCH` | `/api/auth/user/update` | Session Cookie | Updates `first_name` and/or `last_name`. Body: `{ first_name?, last_name? }`. |

### Post Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/posts/create?project_id=` | Session Cookie | Create a post. Body: `{ title, author, body, teaser, image?, keywords? }`. Validates plan body length limit. Returns `ApiResponse<Post>`. |
| `PATCH` | `/api/posts/update?project_id=&post_id=` | Session Cookie | Partially update a post. Body: any subset of `{ title, author, body, teaser, keywords, image }`. |
| `DELETE` | `/api/posts/delete?project_id=&post_id=` | Session Cookie | Delete a post by pulling it from the project's `posts` array. |
| `GET` | `/api/posts/get/single?post_id=` | Bearer `project_key` | Fetch a single full post (used by SDK). Appends Atom watermark to `body` for `single` plan users. |

### Project Routes

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/projects/create` | Session Cookie | Create a project. Body: `{ title }`. Validates plan's `max_projects` limit. Creates `Project` + updates `UserDocument.projects` in a transaction. Returns `ApiResponse<Project>`. |
| `DELETE` | `/api/projects/delete?project_id=` | Session Cookie | Delete project + remove from `UserDocument.projects` in a transaction. |
| `GET` | `/api/projects/get/single?project_id=` | Session Cookie OR Bearer `project_key` | Fetch full project (with post bodies). If `Authorization: Bearer` header present, authenticates by project key; otherwise uses session + `project_id` query param. |
| `GET` | `/api/projects/get/single/client` | Bearer `project_key` | Public endpoint used by the SDK. Returns `ClientProject` (posts without `body` — only metadata fields safe for public listing). |

---

## Dependencies

### Main Application

| Dependency | Purpose |
|-----------|---------|
| `next` 14.1.0 | Framework (App Router) |
| `lucia` ^3.1.1 | Session-based authentication |
| `@lucia-auth/adapter-mongodb` ^1.0.2 | Lucia adapter for MongoDB |
| `mongoose` ^8.1.2 | MongoDB ODM |
| `@upstash/redis` + `@upstash/ratelimit` | Redis client + sliding window rate limiting |
| `argon2` ^0.40.1 | Password hashing |
| `uuid` ^9.0.1 | UUID generation for user and project IDs |
| `zod` ^3.22.4 | Schema validation (forms + API input types) |
| `react-hook-form` + `@hookform/resolvers` | Form state management with Zod integration |
| `@tanstack/react-table` | Headless table (projects dashboard) |
| `@uiw/react-md-editor` | Markdown editor UI component |
| `next-mdx-remote` | MDX rendering (via SDK's AtomBody) |
| `axios` ^1.6.7 | HTTP client in `lib/client/**` functions |
| `@radix-ui/*` | Headless UI primitives (used by shadcn/ui) |
| `tailwind-merge` + `clsx` | Utility for conditional Tailwind classes (`cn()`) |
| `framer-motion` | Animation (tracing-beam component) |
| `react-hot-toast` | Toast notifications |
| `react-icons` | Icon library |
| `react-tweet` | Embed tweets (currently commented out in home page) |
| `atom-nextjs` ^0.3.1 | Own SDK (dog-fooded for the app's blog page) |

### SDK (`atom-nextjs`)

| Dependency | Purpose |
|-----------|---------|
| `next` | Next.js server components + Metadata API |
| `next-mdx-remote` | `compileMDX` for rendering Markdown/MDX |
| `react-loading-skeleton` | Loading skeleton UI |
| `remark-gfm` | GitHub-flavored Markdown support |
| `rehype-sanitize` | HTML sanitization of MDX output |
| `tsdx` | Build toolchain for the npm package |

---

## Build & Run

### Environment Variables (`.env.local`)
```bash
HASH_SALT="<random secret>"             # Salt appended to passwords before argon2 hashing
MONGO_DB_URI="<mongodb+srv://...>"      # MongoDB connection string
ATOM_PROJECT_KEY="<atom-...>"          # Project key for the app's own blog
UPSTASH_REDIS_REST_URL="<url>"          # Upstash Redis URL
UPSTASH_REDIS_REST_TOKEN="<token>"      # Upstash Redis token
NEXT_PUBLIC_ENV="dev"                   # "dev" or "prod" (controls baseAPIRoute)
```

### Main Application
```bash
# Install dependencies (project root)
npm install

# Development server
npm run dev        # → http://localhost:3000

# Production build
npm run build

# Start production server
npm run start

# Lint
npm run lint
```

### SDK (`packages/atom-nextjs/`)
```bash
cd packages/atom-nextjs
npm install

# Build the package (tsdx)
npm run build      # or: npm run prepare

# Watch mode
npm run start

# Test
npm run test

# Local linking for development
npm link
# Then back in project root:
npm link atom-nextjs
```

### Running SDK Locally with the App
1. `cd packages/atom-nextjs && npm link`
2. `cd ../.. && npm link atom-nextjs`
3. Restart code editor if TypeScript doesn't resolve types
4. The app's `baseAPIRoute` in `lib/contants.tsx` auto-switches to `http://localhost:3000/api` when `NEXT_PUBLIC_ENV` is not `"prod"`.

---

## Patterns & Conventions

### API Response Shape
Every API route returns `ApiResponse<T>` from `app/api/auth/signup/route.ts`:
```ts
{ success: boolean; message: string | null; response: T }
```
Errors are always caught in a try/catch and returned as `success: false` with `message: err.message || err`. Client wrappers in `lib/client/**` throw if `!data.success`, propagating the message to `react-hot-toast`.

### Error Handling
- Server: `try/catch` in every route handler; never throw unhandled. Known error codes (e.g., MongoDB duplicate key `11000`) are handled explicitly.
- Client: `lib/client/**` functions throw `new Error(data.message)` if `!success`; components catch and call `toast.error(err.message || err)`.
- `@ts-expect-error` is used in a few places where TypeScript cannot infer correctly (e.g., Mongoose `.create()` returns `never` in some contexts, Lucia session attributes).

### Authentication Pattern
1. Every protected server component/route calls `await connectToDatabase()` first, then `await validateRequest()`.
2. `validateRequest()` is wrapped in React `cache()` to deduplicate calls within a single render cycle.
3. Ownership checks (`project.creator_uid === user.id`) are performed in every project/post mutation route.

### Database Transactions
Multi-document writes (signup, delete user, create/delete project) use `mongoose.startSession()` + `mongooseSession.withTransaction(async () => { ... })` to ensure atomicity.

### Form Validation
All forms use `react-hook-form` with `zodResolver`. Zod schemas are defined inline in the component file and their inferred types exported (e.g., `LoginFormInputs`, `SignupFormInputs`, `ProjectFormInputs`). API-level validation mirrors the Zod constraints in the route handler.

### Path Aliases
`@/*` maps to the project root (e.g., `@/lib/types`, `@/components/ui/button`).

### Component Organization
- **Server components** by default (no directive) — all layout, data-fetching, and container components.
- **Client components** marked with `"use client"` at the top — all form, modal, interactive, and sidebar components.
- UI primitives live in `components/ui/` and are generated/configured via shadcn/ui.

### Naming Conventions
- Files use `camelCase` for utilities (e.g., `generateProjectKey.ts`), `PascalCase` for React components (e.g., `ProjectPage.tsx`).
- API route files are always named `route.ts`.
- Client API wrapper functions are named after the action (e.g., `createPost`, `deleteProject`).
- Mongoose model refs are suffixed `Ref` (e.g., `UserCredentialsRef`, `ProjectsRef`).
- Request body types exported from route files are named `<Action>RequestParams` or `<Action>RequestBody`.

### Rate Limiting
Only `/api/**` routes are rate-limited (via `middleware.ts` matcher). Limit is 30 requests per sliding 1-minute window per IP using Upstash Redis. Exceeding the limit returns `ApiResponse` with `success: false, message: "Too many requests."`.

### Plan Enforcement
Plan limits are enforced server-side in the relevant route handlers:
- `POST /api/projects/create` — checks `userPlan.max_projects <= userDoc.projects.length`.
- `POST /api/posts/create` and `PATCH /api/posts/update` — checks `data.body.length > userPlan.max_body_length`.
- `GET /api/posts/get/single` — appends watermark to `body` if user plan is `"single"`.
