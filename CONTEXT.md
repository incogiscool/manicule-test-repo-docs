Now I have a comprehensive understanding of the codebase. Let me produce the CONTEXT.md:

# Atom CMS

## Overview
Atom is a Content Management System (CMS) built specifically for Next.js projects. It allows users to create, edit, and publish blog posts through a web dashboard, then render them in their own Next.js apps via the companion `atom-nextjs` npm SDK. The tech stack is **Next.js 14** (App Router), **TypeScript**, **MongoDB** (via Mongoose), **Lucia** for authentication, **Argon2** for password hashing, **Upstash Redis** for API rate limiting, **TailwindCSS** with **shadcn/ui** components, **React Hook Form** + **Zod** for form validation, and **Axios** for client-side HTTP. The project is a monorepo containing both the main web app and the `atom-nextjs` SDK package.

## File Tree
```
/
├── .eslintrc.json
├── .gitignore
├── LICENSE                          # Apache 2.0
├── README.md
├── bun.lock
├── components.json                  # shadcn/ui config
├── middleware.ts                     # Rate limiting middleware
├── next.config.mjs
├── package.json
├── package-lock.json
├── postcss.config.js
├── tailwind.config.ts
├── tsconfig.json
├── public/
│   ├── atom-black.svg
│   ├── next.svg
│   └── vercel.svg
├── app/
│   ├── favicon.ico
│   ├── globals.css                  # Tailwind base + CSS variables
│   ├── layout.tsx                   # Root layout (Montserrat font, Toaster)
│   ├── page.tsx                     # Landing/marketing page
│   ├── robots.ts                    # SEO robots.txt
│   ├── sitemap.ts                   # Dynamic sitemap using atom-nextjs SDK
│   ├── signin/
│   │   └── page.tsx
│   ├── signup/
│   │   └── page.tsx
│   ├── pricing/
│   │   └── page.tsx
│   ├── blog/
│   │   ├── page.tsx                 # Public blog listing (uses atom-nextjs SDK)
│   │   └── [id]/
│   │       └── page.tsx             # Public blog post (uses atom-nextjs SDK)
│   ├── app/
│   │   ├── layout.tsx               # Protected route wrapper
│   │   ├── page.tsx                 # Dashboard: project list
│   │   ├── projects/
│   │   │   └── [id]/
│   │   │       └── page.tsx         # Project detail: post editor
│   │   └── settings/
│   │       ├── page.tsx             # User settings
│   │       └── billing/
│   │           └── page.tsx         # Billing (coming soon)
│   └── api/
│       ├── auth/
│       │   ├── signup/route.ts      # POST - create account
│       │   ├── signin/route.ts      # POST - login
│       │   ├── signout/route.ts     # POST - logout
│       │   ├── delete/route.ts      # DELETE - delete account
│       │   └── user/
│       │       ├── get/route.ts     # GET - get current user
│       │       └── update/route.ts  # (referenced but file missing)
│       ├── posts/
│       │   ├── create/route.ts      # POST - create post
│       │   ├── delete/route.ts      # DELETE - delete post
│       │   ├── update/route.ts      # PATCH - update post
│       │   └── get/
│       │       └── single/route.ts  # GET - get post by key+id (public)
│       └── projects/
│           ├── create/route.ts      # POST - create project
│           ├── delete/route.ts      # DELETE - delete project
│           └── get/
│               └── single/
│                   ├── route.ts     # GET - get project (auth or key)
│                   └── client/route.ts # GET - get project for client SDK
├── components/
│   ├── cards/
│   │   └── PricingPlanCard.tsx
│   ├── containers/
│   │   ├── AppContainer.tsx         # Dashboard shell with sidebar
│   │   ├── MainContainer.tsx        # Marketing page shell with navbar+footer
│   │   └── ProtectedRoute.tsx       # Server-side auth guard
│   ├── forms/
│   │   ├── LoginForm.tsx
│   │   └── SignupForm.tsx
│   ├── misc/
│   │   ├── NpmPackageComponent.tsx  # npm install copy widget
│   │   └── tracing-beam.tsx         # Aceternity UI component
│   ├── modals/
│   │   ├── CreatePostModal.tsx
│   │   └── DeleteUserModal.tsx
│   ├── nav/
│   │   └── Navbar.tsx               # Marketing nav (server component)
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectComponent.tsx # Project detail with post sidebar+editor
│   │   │   ├── ProjectFormComponent.tsx # Post edit form
│   │   │   └── ProjectPage.tsx      # Dashboard projects list
│   │   └── settings/
│   │       └── SettingsForm.tsx
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx        # Dashboard sidebar navigation
│   │   └── ProjectComponentSidebar.tsx # Post list sidebar in project view
│   ├── tables/
│   │   └── UserDocumentProjects/
│   │       ├── columns.tsx          # TanStack Table column defs
│   │       └── table.tsx            # TanStack Table component
│   └── ui/                          # shadcn/ui primitives
│       ├── alert-dialog.tsx
│       ├── button.tsx
│       ├── carousel.tsx
│       ├── dialog.tsx
│       ├── dropdown-menu.tsx
│       ├── form.tsx
│       ├── input.tsx
│       ├── label.tsx
│       ├── markdown-editor.tsx      # @uiw/react-md-editor wrapper
│       ├── popover.tsx
│       ├── sticky-scroll-reveal.tsx
│       ├── table.tsx
│       └── textarea.tsx
├── lib/
│   ├── contants.tsx                 # Plans, nav options, baseAPIRoute, limits
│   ├── types.ts                     # Core TypeScript types
│   ├── utils.ts                     # cn() from shadcn/ui
│   ├── utils/
│   │   └── validateEmail.ts
│   ├── client/                      # Client-side API call wrappers
│   │   ├── auth/
│   │   │   ├── deleteUser.ts
│   │   │   ├── loginUser.ts
│   │   │   ├── signoutUser.ts
│   │   │   ├── signupUser.ts
│   │   │   └── updateUser.ts
│   │   ├── posts/
│   │   │   ├── createPost.ts
│   │   │   ├── deletePost.ts
│   │   │   └── updatePost.ts
│   │   └── projects/
│   │       ├── createProject.ts
│   │       └── deleteProject.ts
│   └── server/                      # Server-only utilities
│       ├── encoding/
│       │   ├── encodePassword.ts    # Argon2 hash with salt
│       │   └── isPasswordValid.ts   # Argon2 verify
│       ├── functions/
│       │   ├── projects/
│       │   │   └── getProject.ts    # Server-side project fetch via cookies
│       │   └── user/
│       │       └── fetchUser.ts     # Server-side user fetch via cookies
│       ├── lucia/
│       │   ├── init.ts              # Lucia auth + MongoDB adapter setup
│       │   └── functions/
│       │       └── validate-request.ts # Cached session validation
│       ├── mongo/
│       │   ├── init.ts              # Mongoose connection + model refs
│       │   └── types/
│       │       ├── userCredentials.ts
│       │       ├── userDocuments.ts
│       │       ├── userProjects.ts
│       │       └── userSessions.ts
│       ├── redis/
│       │   └── init.ts             # Upstash Redis + rate limiter
│       └── utils/
│           ├── generateProjectKey.ts
│           ├── validateProjectKey.ts
│           └── validateRequestFetchUser.ts
├── bruno/                           # Bruno API client collection
│   ├── bruno.json
│   ├── environments/
│   │   └── Development.bru
│   └── Routes/
│       ├── Create Post.bru
│       ├── Create Project.bru
│       ├── Delete Post.bru
│       ├── Delete Project.bru
│       ├── Get Post.bru
│       ├── Get Project.bru
│       └── Get User.bru
└── packages/
    └── atom-nextjs/                 # Published npm SDK package
        ├── .github/workflows/
        │   ├── main.yml
        │   └── size.yml
        ├── .gitignore
        ├── LICENSE
        ├── README.md                # SDK usage documentation
        ├── package.json
        ├── package-lock.json
        ├── tsconfig.json
        ├── yarn.lock
        └── src/
            ├── index.tsx            # Package entry — re-exports all
            ├── components/
            │   ├── Atom.tsx         # Single post renderer (SSR)
            │   ├── AtomArticleSkeleton.tsx
            │   ├── AtomBody.tsx     # MDX body renderer
            │   ├── AtomLoadingSkeleton.tsx
            │   ├── AtomPage.tsx     # Blog listing page (SSR)
            │   └── AtomPostCard.tsx # Post card component
            └── lib/
                ├── constants.ts     # API base URL
                ├── types.ts         # SDK types
                └── client/
                    ├── generatePostMetadata.ts
                    ├── generateSitemap.ts
                    ├── getPost.ts
                    └── getProject.ts
```

## Architecture

### High-Level Architecture

Atom follows a **Next.js 14 App Router** architecture with a clear separation between:

1. **Marketing/Public pages** (`app/page.tsx`, `app/pricing/`, `app/blog/`, `app/signin/`, `app/signup/`) — rendered with the `MainContainer` shell (Navbar + footer).
2. **Dashboard/App pages** (`app/app/`) — protected by `ProtectedRoute` (server-side auth check), rendered with `AppContainer` shell (sidebar navigation).
3. **API routes** (`app/api/`) — RESTful JSON endpoints for auth, posts, and projects.
4. **SDK package** (`packages/atom-nextjs/`) — a standalone npm library consumers install to render blog content from Atom in their own Next.js apps.

### Data Flow

1. **Authentication**: Users sign up/in via forms → client calls `lib/client/auth/*` → hits `app/api/auth/*` → Mongoose stores credentials + user documents in MongoDB → Lucia creates session cookies.
2. **Project Management**: Authenticated users create/delete projects via dashboard → client calls `lib/client/projects/*` → hits `app/api/projects/*` → Mongoose CRUD on `projects` and `documents` collections (atomic via MongoDB transactions).
3. **Post Management**: Within a project, users create/edit/delete posts → client calls `lib/client/posts/*` → hits `app/api/posts/*` → Posts are stored as subdocuments within the `projects` collection.
4. **Public Blog Rendering**: The `atom-nextjs` SDK fetches project/post data via Bearer token (project key) from the public API endpoints, renders MDX content server-side.
5. **Rate Limiting**: All `/api/*` routes are rate-limited via middleware using Upstash Redis (30 requests per minute per IP).

### Module Responsibilities

| Directory | Purpose |
|---|---|
| `app/api/` | REST API endpoints (auth, posts, projects) |
| `app/app/` | Authenticated dashboard pages |
| `app/blog/` | Public blog pages using atom-nextjs SDK |
| `components/` | React components (containers, forms, modals, sidebars, tables, ui) |
| `lib/client/` | Client-side API call wrappers using Axios |
| `lib/server/` | Server-only: auth (Lucia), DB (Mongoose), rate limiting (Redis), encoding |
| `lib/types.ts` | Shared TypeScript type definitions |
| `lib/contants.tsx` | App constants (plans, nav options, limits, base URL) |
| `packages/atom-nextjs/` | Published npm SDK for end-users |

## Key Files

### Root Configuration
- **`package.json`** — Project manifest for "atom"; declares Next.js 14, Lucia, Mongoose, shadcn/ui dependencies.
- **`next.config.mjs`** — Disables caching (`no-store`) for `/`, `/app/*`, and `/api/*` routes.
- **`middleware.ts`** — Applies Upstash rate limiting (30 req/min) to all `/api/*` routes.
- **`tailwind.config.ts`** — Tailwind config with shadcn/ui theme, `@tailwindcss/typography`, and a custom plugin that exposes all Tailwind colors as CSS variables.
- **`components.json`** — shadcn/ui configuration (default style, RSC, TSX, slate base color).
- **`tsconfig.json`** — TypeScript config with `@/*` path alias mapping to project root.

### App Entry Points
- **`app/layout.tsx`** — Root layout: Montserrat font, `react-hot-toast` Toaster, HTML skeleton.
- **`app/page.tsx`** — Marketing landing page with hero, code examples, demo iframe, CTA.
- **`app/app/layout.tsx`** — Wraps all dashboard pages in `ProtectedRoute` (redirects unauthenticated users).
- **`app/app/page.tsx`** — Dashboard home: fetches user server-side, renders `ProjectPage` with projects table.
- **`app/app/projects/[id]/page.tsx`** — Project detail: fetches project server-side, renders `ProjectComponent` (post sidebar + editor).

### Authentication
- **`app/api/auth/signup/route.ts`** — `POST`: Creates user credentials + user document in MongoDB transaction, starts Lucia session. Defines `ApiResponse<T>` type used across all APIs.
- **`app/api/auth/signin/route.ts`** — `POST`: Validates email/password, creates Lucia session.
- **`app/api/auth/signout/route.ts`** — `POST`: Invalidates Lucia session, clears cookie.
- **`app/api/auth/delete/route.ts`** — `DELETE`: Deletes user credentials, documents, projects, sessions in MongoDB transaction.
- **`app/api/auth/user/get/route.ts`** — `GET`: Returns authenticated user's document.
- **`lib/server/lucia/init.ts`** — Initializes Lucia auth with MongoDB adapter, configures session cookies.
- **`lib/server/lucia/functions/validate-request.ts`** — Cached server-side session validation using `cookies()`.
- **`lib/server/encoding/encodePassword.ts`** — Hashes passwords with Argon2 + environment salt.
- **`lib/server/encoding/isPasswordValid.ts`** — Verifies passwords against Argon2 hashes.

### Posts API
- **`app/api/posts/create/route.ts`** — `POST`: Creates a post as a subdocument in a project. Validates plan body length limits.
- **`app/api/posts/update/route.ts`** — `PATCH`: Updates post fields using MongoDB positional operator (`posts.$.field`).
- **`app/api/posts/delete/route.ts`** — `DELETE`: Removes post from project's posts array via `$pull`.
- **`app/api/posts/get/single/route.ts`** — `GET`: Public endpoint, fetches single post by Bearer project key + post_id. Appends watermark for free plan.

### Projects API
- **`app/api/projects/create/route.ts`** — `POST`: Creates project + updates user document in MongoDB transaction. Enforces plan project limits.
- **`app/api/projects/delete/route.ts`** — `DELETE`: Deletes project + removes from user document in transaction.
- **`app/api/projects/get/single/route.ts`** — `GET`: Dual-mode — fetches project by Bearer key (public) or by session + project_id (authenticated).
- **`app/api/projects/get/single/client/route.ts`** — `GET`: Public endpoint for SDK. Returns sanitized `ClientProject` (no sensitive fields).

### Database
- **`lib/server/mongo/init.ts`** — Mongoose connection + model references (`UserCredentialsRef`, `UserDocumentsRef`, `ProjectsRef`, `SessionRef`).
- **`lib/server/mongo/types/userCredentials.ts`** — Mongoose schema: email, password_hash, _id (UUID).
- **`lib/server/mongo/types/userDocuments.ts`** — Mongoose schema: first_name, last_name, email, plan, projects (embedded).
- **`lib/server/mongo/types/userProjects.ts`** — Mongoose schemas: `postSchema` (subdocument), `projectsSchema` (title, posts array, project_key, creator_uid).
- **`lib/server/mongo/types/userSessions.ts`** — Mongoose schema for Lucia sessions.

### Client-Side Helpers
- **`lib/client/auth/*`** — Axios wrappers for auth API calls (login, signup, signout, delete, update).
- **`lib/client/posts/*`** — Axios wrappers for post CRUD (create, update, delete). Show toast on success.
- **`lib/client/projects/*`** — Axios wrappers for project create/delete. Show toast on success.
- **`lib/server/functions/user/fetchUser.ts`** — Server-side helper that calls the user/get API using forwarded cookies.
- **`lib/server/functions/projects/getProject.ts`** — Server-side helper that calls the project/get API using forwarded cookies.

### SDK Package (`packages/atom-nextjs/`)
- **`src/index.tsx`** — Re-exports all public SDK exports: `Atom`, `AtomBody`, `AtomPage`, `AtomPostCard`, `AtomLoadingSkeleton`, `AtomArticleSkeleton`, `generatePostMetadata`, `getPost`, `getProject`, `generateSitemap`.
- **`src/components/Atom.tsx`** — Server component that fetches a single post and renders it with header, image, metadata, and MDX body.
- **`src/components/AtomPage.tsx`** — Server component that fetches all posts for a project and renders post cards.
- **`src/components/AtomBody.tsx`** — MDX renderer using `next-mdx-remote/rsc` with remark-gfm and rehype-sanitize.
- **`src/components/AtomPostCard.tsx`** — Post card with image, date, author, title, teaser.
- **`src/lib/client/getProject.ts`** — Fetches project from Atom API using Bearer project key.
- **`src/lib/client/getPost.ts`** — Fetches single post from Atom API using Bearer project key.
- **`src/lib/client/generatePostMetadata.ts`** — Generates Next.js `Metadata` object for a post (title, description, keywords, author).
- **`src/lib/client/generateSitemap.ts`** — Generates Next.js sitemap entries for all posts in a project.

### Components
- **`components/containers/ProtectedRoute.tsx`** — Server component that validates session and redirects to `/signin` if unauthenticated.
- **`components/containers/AppContainer.tsx`** — Dashboard layout shell with `AppSidebarNav` + main content area.
- **`components/containers/MainContainer.tsx`** — Marketing layout shell with `Navbar` + footer.
- **`components/pages/projects/ProjectComponent.tsx`** — Client component managing post selection state and rendering sidebar + form editor.
- **`components/pages/projects/ProjectFormComponent.tsx`** — Post edit form with Zod validation, markdown editor, delete confirmation dialog.
- **`components/ui/markdown-editor.tsx`** — Wrapper around `@uiw/react-md-editor` with rehype-sanitize.

## Data Model

### MongoDB Collections & Schemas

```
credentials (UserCredentials)
├── _id: string (UUID)
├── email: string (unique, lowercase, trimmed)
├── password_hash: string
├── createdAt: Date (auto)
└── updatedAt: Date (auto)

documents (UserDocument)
├── _id: string (UUID, same as credentials._id)
├── first_name: string
├── last_name: string
├── email: string
├── plan: "single" | "startup" | "business"
├── projects: UserDocumentProjects[]  (embedded)
│   ├── id: string
│   ├── title: string
│   ├── createdAt: Date
│   ├── updatedAt: Date
│   └── creator: { uid: string, email: string }
├── createdAt: Date (auto)
└── updatedAt: Date (auto)

projects (Project)
├── _id: string (UUID)
├── title: string
├── project_key: string (base64 random, prefixed "atom-")
├── creator_uid: string
├── posts: Post[]  (embedded subdocuments)
│   ├── id: string (UUID)
│   ├── title: string
│   ├── author: string
│   ├── body: string (markdown)
│   ├── image: string | null
│   ├── keywords: string[]
│   ├── teaser: string
│   ├── creator_uid: string
│   ├── createdAt: Date (auto)
│   └── updatedAt: Date (auto)
├── createdAt: Date (auto)
└── updatedAt: Date (auto)

sessions (Session - managed by Lucia)
├── _id: ObjectId (auto)
├── user_id: string
└── expires_at: Date
```

### Relationships
- `credentials._id` ↔ `documents._id` — 1:1 (same UUID)
- `documents.projects[].id` → `projects._id` — denormalized reference
- `projects.creator_uid` → `credentials._id` / `documents._id`
- `sessions.user_id` → `credentials._id`

### Plan Limits
| Plan | Price | Max Projects | Max Posts | Max Body Length |
|------|-------|-------------|-----------|-----------------|
| single | Free | 2 | 100 | 10,000 chars |
| startup | $3.99/mo | 3 | 1,000 | 100,000 chars |
| business | $11.99/mo | 5 | 2,500 | 500,000 chars |

*Note: startup and business plans are currently disabled (`disabled: true`).*

## API / Routes

All API routes return `ApiResponse<T>`:
```ts
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

### Auth Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/auth/signup` | None | Create account. Body: `{ email, password, first_name, last_name }`. Returns `ApiResponse<UserDocument>`. |
| `POST` | `/api/auth/signin` | None | Login. Body: `{ email, password }`. Returns `ApiResponse<UserDocument>`. |
| `POST` | `/api/auth/signout` | Session cookie | Logout. Invalidates session. |
| `DELETE` | `/api/auth/delete` | Session cookie | Delete account. Body: `{ password }`. Deletes all user data in transaction. |
| `GET` | `/api/auth/user/get` | Session cookie | Get current user document. Returns `ApiResponse<UserDocument>`. |

### Post Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/posts/create?project_id=X` | Session cookie | Create post. Body: `{ title, author, body, image?, keywords?, teaser }`. Returns `ApiResponse<Post>`. |
| `PATCH` | `/api/posts/update?project_id=X&post_id=Y` | Session cookie | Update post. Body: `{ title?, author?, body?, teaser?, keywords?, image? }`. |
| `DELETE` | `/api/posts/delete?project_id=X&post_id=Y` | Session cookie | Delete post. |
| `GET` | `/api/posts/get/single?post_id=X` | Bearer project_key | Get single post (public). Appends watermark for free plan. Returns `ApiResponse<Post>`. |

### Project Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/api/projects/create` | Session cookie | Create project. Body: `{ title }`. Enforces plan limits. Returns `ApiResponse<Project>`. |
| `DELETE` | `/api/projects/delete?project_id=X` | Session cookie | Delete project and remove from user document. |
| `GET` | `/api/projects/get/single?project_id=X` | Session cookie OR Bearer project_key | Get full project. |
| `GET` | `/api/projects/get/single/client` | Bearer project_key | Get sanitized project for SDK clients. Returns `ApiResponse<ClientProject>`. |

### Page Routes

| Path | Description |
|------|-------------|
| `/` | Marketing landing page |
| `/signin` | Login page |
| `/signup` | Registration page |
| `/pricing` | Pricing plans page |
| `/blog` | Public blog listing (uses atom-nextjs SDK) |
| `/blog/[id]` | Public blog post (uses atom-nextjs SDK) |
| `/app` | Dashboard: project list (protected) |
| `/app/projects/[id]` | Project detail: post editor (protected) |
| `/app/settings` | User settings (protected) |
| `/app/settings/billing` | Billing page — coming soon (protected) |

## Dependencies

### Main App
| Package | Purpose |
|---------|---------|
| `next` 14.1.0 | React framework (App Router) |
| `lucia` 3.1.1 | Session-based authentication |
| `@lucia-auth/adapter-mongodb` | Lucia adapter for MongoDB |
| `mongoose` 8.1.2 | MongoDB ODM |
| `argon2` 0.40.1 | Password hashing |
| `@upstash/redis` + `@upstash/ratelimit` | API rate limiting |
| `axios` 1.6.7 | HTTP client for client-side API calls |
| `react-hook-form` + `@hookform/resolvers` | Form management |
| `zod` 3.22.4 | Schema validation |
| `@tanstack/react-table` 8.13.2 | Data table for project listing |
| `@uiw/react-md-editor` 4.0.4 | Markdown editor for post body |
| `react-hot-toast` 2.4.1 | Toast notifications |
| `react-markdown` + `react-syntax-highlighter` | Markdown rendering on landing page |
| `@radix-ui/*` | Primitives for shadcn/ui components |
| `tailwind-merge` + `clsx` + `class-variance-authority` | Utility-first CSS composition |
| `tailwindcss-animate` | Tailwind animation utilities |
| `@tailwindcss/typography` | Prose styling for blog content |
| `framer-motion` 11.0.15 | Animations (used in UI components) |
| `zustand` 4.5.0 | State management (declared but not actively used in visible code) |
| `uuid` 9.0.1 | UUID generation for IDs |
| `atom-nextjs` 0.3.1 | Own SDK (self-consuming for the `/blog` pages) |
| `react-tweet` 3.2.0 | Twitter embed components |
| `oslo` 1.1.1 | Cryptographic utilities (Lucia dependency) |

### SDK Package (`atom-nextjs`)
| Package | Purpose |
|---------|---------|
| `next` ^14.2.15 | Peer/runtime dependency |
| `next-mdx-remote` 4.4.1 | Server-side MDX compilation |
| `react-loading-skeleton` 3.5.0 | Loading skeleton components |
| `rehype-sanitize` 6.0.0 | HTML sanitization for MDX |
| `remark-gfm` 3.0.0 | GitHub-flavored Markdown support |
| `tsdx` 0.14.1 | Build/test/lint toolchain |

## Build & Run

### Prerequisites
- Node.js >= 10 (SDK), recommended 18+ for main app
- MongoDB instance
- Upstash Redis instance

### Environment Variables
```bash
HASH_SALT="..."              # Salt appended to passwords before Argon2 hashing
MONGO_DB_URI="..."           # MongoDB connection string
ATOM_PROJECT_KEY="..."       # Project key for own blog pages
UPSTASH_REDIS_REST_URL="..." # Upstash Redis URL
UPSTASH_REDIS_REST_TOKEN="..." # Upstash Redis token
NEXT_PUBLIC_ENV="dev"        # "dev" or "prod" — controls baseAPIRoute
ENV="dev"                    # "dev" or "prod"
```

### Scripts (Main App)
```bash
npm run dev      # Start Next.js dev server
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # ESLint
```

### Scripts (SDK — `packages/atom-nextjs/`)
```bash
npm run start    # tsdx watch (development)
npm run build    # tsdx build
npm run test     # tsdx test
npm run lint     # tsdx lint
```

### Local SDK Development
```bash
cd packages/atom-nextjs
npm link
cd ../..
npm link atom-nextjs
```

## Patterns & Conventions

### Code Organization
- **Path alias**: `@/*` maps to the project root. All imports use this (e.g., `@/lib/types`, `@/components/ui/button`).
- **File naming**: Files use camelCase for utilities/functions, PascalCase for React components. Note: `lib/contants.tsx` is a typo of "constants" — maintained throughout the codebase.
- **Component directive**: Client components are explicitly marked with `"use client"`. Server components (default) have no directive.

### API Pattern
- All API routes follow a consistent pattern: try/catch with `NextResponse.json<ApiResponse<T>>()`.
- All API routes call `connectToDatabase()` before any DB operation.
- Authenticated routes call `validateRequest()` to get the session/user.
- Authorization checks compare `user.id` against resource `creator_uid`.
- Multi-document mutations use MongoDB transactions (`mongoose.startSession()` + `withTransaction()`).

### Authentication
- Session-based auth via Lucia v3 with MongoDB adapter.
- Session validation is cached per request using React's `cache()`.
- Protected pages use `ProtectedRoute` server component that redirects to `/signin`.

### Forms
- All forms use `react-hook-form` + `zod` for validation via `@hookform/resolvers`.
- Form schemas are defined inline or in the same file as the component.
- Loading state managed via `useState(isLoading)`.

### Error Handling
- API routes: try/catch blocks, errors returned as `{ success: false, message: err.message }`.
- Client-side: try/catch blocks, errors displayed via `toast.error()`.
- No global error boundary observed.

### Styling
- TailwindCSS with shadcn/ui (default style, slate base color, CSS variables).
- `cn()` utility (clsx + tailwind-merge) used for conditional class merging.
- Light theme only (dark mode defined in CSS but not actively toggled).

### State Management
- Server components fetch data server-side (using forwarded cookies for auth).
- Client components receive data as props from server components.
- Local state via `useState`; `router.refresh()` used to re-fetch server data after mutations.

### SDK Design
- All SDK components are **async server components** to protect the project API key.
- MDX is compiled server-side via `next-mdx-remote/rsc`.
- SDK uses `fetch` (not Axios) for API calls.
- SDK is built with `tsdx` and published as `atom-nextjs` on npm.
