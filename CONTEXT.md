Now I have a thorough understanding of the entire codebase. Let me produce the CONTEXT.md:

# Atom CMS

## Overview
Atom is a Content Management System (CMS) built for Next.js projects, enabling users to create, edit, and publish blogs/articles through a web dashboard and consume them via an SDK (`atom-nextjs`). The tech stack is **Next.js 14** (App Router), **TypeScript**, **MongoDB** (via Mongoose), **Lucia** for authentication, **Upstash Redis** for rate limiting, **Tailwind CSS** + **shadcn/ui** for the UI, and **Zod** + **react-hook-form** for form validation. The project also includes a companion npm package (`atom-nextjs`) that provides React components for rendering blog pages on consumer sites.

## File Tree
```
context/
├── randomcode.ts                          # Trivial console.log placeholder
└── test-code/
    ├── .eslintrc.json
    ├── .gitignore
    ├── LICENSE                            # Apache 2.0
    ├── README.md
    ├── bun.lock
    ├── components.json                    # shadcn/ui config
    ├── middleware.ts                       # Rate limiting middleware for /api routes
    ├── next.config.mjs                    # Next.js config (Cache-Control headers)
    ├── package.json
    ├── package-lock.json
    ├── postcss.config.js
    ├── tailwind.config.ts
    ├── tsconfig.json
    │
    ├── app/
    │   ├── favicon.ico
    │   ├── globals.css                    # Tailwind + CSS variables (shadcn theme)
    │   ├── layout.tsx                     # Root layout (Montserrat font, Toaster)
    │   ├── page.tsx                       # Landing / marketing homepage
    │   ├── robots.ts                      # SEO robots.txt
    │   ├── sitemap.ts                     # Dynamic sitemap via atom-nextjs SDK
    │   │
    │   ├── signin/
    │   │   └── page.tsx                   # Sign-in page
    │   ├── signup/
    │   │   └── page.tsx                   # Sign-up page
    │   ├── pricing/
    │   │   └── page.tsx                   # Pricing plans page
    │   ├── blog/
    │   │   ├── page.tsx                   # Blog listing (uses AtomPage from SDK)
    │   │   └── [id]/
    │   │       └── page.tsx               # Single blog post (uses Atom from SDK)
    │   │
    │   ├── app/                           # Authenticated dashboard area
    │   │   ├── layout.tsx                 # ProtectedRoute wrapper
    │   │   ├── page.tsx                   # Projects list (dashboard home)
    │   │   ├── projects/
    │   │   │   └── [id]/
    │   │   │       └── page.tsx           # Single project view (post editor)
    │   │   └── settings/
    │   │       ├── page.tsx               # User settings
    │   │       └── billing/
    │   │           └── page.tsx           # Billing (placeholder)
    │   │
    │   └── api/
    │       ├── auth/
    │       │   ├── signup/route.ts        # POST - user registration
    │       │   ├── signin/route.ts        # POST - user login
    │       │   ├── signout/route.ts       # POST - user logout
    │       │   ├── delete/route.ts        # DELETE - delete user account
    │       │   └── user/
    │       │       ├── get/route.ts       # GET - fetch current user document
    │       │       └── update/route.ts    # PATCH - update user profile
    │       ├── posts/
    │       │   ├── create/route.ts        # POST - create a post in a project
    │       │   ├── delete/route.ts        # DELETE - delete a post
    │       │   ├── update/route.ts        # PATCH - update a post
    │       │   └── get/
    │       │       └── single/route.ts    # GET - fetch single post (by project_key)
    │       └── projects/
    │           ├── create/route.ts        # POST - create a project
    │           ├── delete/route.ts        # DELETE - delete a project
    │           └── get/
    │               └── single/
    │                   ├── route.ts       # GET - fetch project (auth or key)
    │                   └── client/route.ts # GET - public client-safe project data
    │
    ├── components/
    │   ├── cards/
    │   │   └── PricingPlanCard.tsx
    │   ├── containers/
    │   │   ├── AppContainer.tsx           # Dashboard layout shell (sidebar + content)
    │   │   ├── MainContainer.tsx          # Public page layout (navbar + footer)
    │   │   └── ProtectedRoute.tsx         # Server component auth guard
    │   ├── forms/
    │   │   ├── LoginForm.tsx              # Client-side login form (Zod + react-hook-form)
    │   │   └── SignupForm.tsx             # Client-side signup form
    │   ├── misc/
    │   │   ├── NpmPackageComponent.tsx    # Copy-to-clipboard npm install widget
    │   │   └── tracing-beam.tsx           # Aceternity UI tracing beam effect
    │   ├── modals/
    │   │   ├── CreatePostModal.tsx        # Dialog for creating a new post
    │   │   └── DeleteUserModal.tsx        # Dialog for deleting user account
    │   ├── nav/
    │   │   └── Navbar.tsx                 # Public navigation bar (async server component)
    │   ├── pages/
    │   │   ├── projects/
    │   │   │   ├── ProjectPage.tsx        # Projects dashboard with create dialog
    │   │   │   ├── ProjectComponent.tsx   # Project view with sidebar + post editor
    │   │   │   └── ProjectFormComponent.tsx # Post editor form (edit/delete individual post)
    │   │   └── settings/
    │   │       └── SettingsForm.tsx        # User name update form
    │   ├── sidebars/
    │   │   ├── AppSidebarNav.tsx           # Dashboard sidebar (Projects/Billing/Settings)
    │   │   └── ProjectComponentSidebar.tsx # Project sidebar (post list, create, copy key)
    │   ├── tables/
    │   │   └── UserDocumentProjects/
    │   │       ├── columns.tsx            # TanStack Table column defs for projects
    │   │       └── table.tsx              # Generic DataTable component
    │   └── ui/                            # shadcn/ui primitives
    │       ├── alert-dialog.tsx
    │       ├── button.tsx
    │       ├── carousel.tsx
    │       ├── dialog.tsx
    │       ├── dropdown-menu.tsx
    │       ├── form.tsx
    │       ├── input.tsx
    │       ├── label.tsx
    │       ├── markdown-editor.tsx         # MDEditor wrapper (uiw/react-md-editor)
    │       ├── popover.tsx
    │       ├── sticky-scroll-reveal.tsx
    │       ├── table.tsx
    │       └── textarea.tsx
    │
    ├── lib/
    │   ├── contants.tsx                   # App constants (plans, nav, API base URL, limits)
    │   ├── types.ts                       # Core TypeScript types
    │   ├── utils.ts                       # cn() utility (clsx + tailwind-merge)
    │   ├── utils/
    │   │   └── validateEmail.ts           # Email regex validator
    │   ├── client/                        # Client-side API call wrappers (axios)
    │   │   ├── auth/
    │   │   │   ├── loginUser.ts
    │   │   │   ├── signupUser.ts
    │   │   │   ├── signoutUser.ts
    │   │   │   ├── deleteUser.ts
    │   │   │   └── updateUser.ts
    │   │   ├── posts/
    │   │   │   ├── createPost.ts
    │   │   │   ├── deletePost.ts
    │   │   │   └── updatePost.ts
    │   │   └── projects/
    │   │       ├── createProject.ts
    │   │       └── deleteProject.ts
    │   └── server/                        # Server-side only code
    │       ├── encoding/
    │       │   ├── encodePassword.ts      # Argon2 password hashing
    │       │   └── isPasswordValid.ts     # Argon2 password verification
    │       ├── functions/
    │       │   ├── projects/
    │       │   │   └── getProject.ts      # Server-side project fetch (via internal API)
    │       │   └── user/
    │       │       └── fetchUser.ts       # Server-side user fetch (via internal API)
    │       ├── lucia/
    │       │   ├── init.ts                # Lucia auth instance setup
    │       │   └── functions/
    │       │       └── validate-request.ts # Cached session validation
    │       ├── mongo/
    │       │   ├── init.ts                # MongoDB connection + model refs
    │       │   └── types/
    │       │       ├── userCredentials.ts  # Mongoose schema: credentials
    │       │       ├── userDocuments.ts    # Mongoose schema: user documents
    │       │       ├── userProjects.ts     # Mongoose schema: projects & posts
    │       │       └── userSessions.ts     # Mongoose schema: sessions
    │       ├── redis/
    │       │   └── init.ts                # Upstash Redis + rate limiter init
    │       └── utils/
    │           ├── generateProjectKey.ts   # Generates base64 project API key
    │           ├── validateProjectKey.ts   # Validates a project key against DB
    │           └── validateRequestFetchUser.ts # Validates session and fetches user doc
    │
    ├── packages/
    │   └── atom-nextjs/                   # NPM SDK package
    │       ├── package.json
    │       ├── tsconfig.json
    │       ├── yarn.lock
    │       ├── README.md
    │       ├── LICENSE
    │       ├── .gitignore
    │       ├── .github/workflows/
    │       │   ├── main.yml
    │       │   └── size.yml
    │       └── src/
    │           ├── index.tsx              # Package entry (exports all public APIs)
    │           ├── components/
    │           │   ├── Atom.tsx            # Full blog post renderer (server component)
    │           │   ├── AtomBody.tsx        # MDX body renderer (compileMDX)
    │           │   ├── AtomPage.tsx        # Blog listing page component
    │           │   ├── AtomPostCard.tsx    # Post card for listing pages
    │           │   ├── AtomLoadingSkeleton.tsx  # Loading skeleton for AtomPage
    │           │   └── AtomArticleSkeleton.tsx  # Loading skeleton for Atom
    │           └── lib/
    │               ├── constants.ts       # SDK base API route
    │               ├── types.ts           # SDK types (Post, ClientPost, ClientProject)
    │               └── client/
    │                   ├── getPost.ts     # Fetch single post by project key
    │                   ├── getProject.ts  # Fetch project by project key
    │                   ├── generatePostMetadata.ts # Generate Next.js Metadata for post
    │                   └── generateSitemap.ts      # Generate sitemap entries
    │
    └── public/
        ├── atom-black.svg
        ├── next.svg
        └── vercel.svg
```

## Architecture

### High-Level Structure

The project has two main parts:

1. **Main Application** (`context/test-code/`) — A Next.js 14 App Router application serving as:
   - A **marketing site** (landing page, pricing, blog demo)
   - An **authenticated dashboard** where users manage projects and posts
   - A **REST API** for all CRUD operations

2. **SDK Package** (`packages/atom-nextjs/`) — A published npm package providing React server components that consumers install to render blogs on their own Next.js sites.

### Data Flow

```
Consumer Site                    Atom Dashboard                  Atom API
────────────                    ──────────────                  ────────
atom-nextjs SDK ──(Bearer key)──▶ /api/projects/get/single/client ──▶ MongoDB
atom-nextjs SDK ──(Bearer key)──▶ /api/posts/get/single          ──▶ MongoDB
                                 Dashboard UI ──(Session cookie)──▶ /api/* ──▶ MongoDB
```

- **Authentication**: Lucia v3 with MongoDB adapter. Sessions stored in MongoDB `sessions` collection. Cookies managed via `next/headers`.
- **Database**: MongoDB via Mongoose. Four collections: `credentials`, `documents`, `projects`, `sessions`.
- **Rate Limiting**: Upstash Redis with sliding window (30 requests/minute per IP) applied to all `/api/*` routes via middleware.
- **Authorization**: Two models:
  - **Session-based** for dashboard users (cookie → Lucia session validation)
  - **Bearer token** (project_key) for public SDK read-only access to posts/projects

### Directory Roles

| Directory | Role |
|-----------|------|
| `app/` | Next.js App Router pages and API routes |
| `app/api/` | REST API endpoints (auth, posts, projects) |
| `app/app/` | Authenticated dashboard pages (protected by layout) |
| `components/` | React components split by feature (forms, modals, sidebars, pages, ui) |
| `components/ui/` | shadcn/ui primitive components |
| `lib/client/` | Client-side API wrappers using axios |
| `lib/server/` | Server-only code (DB, auth, encoding, utilities) |
| `lib/server/mongo/` | Mongoose connection, schemas, and model references |
| `lib/server/lucia/` | Auth initialization and session validation |
| `packages/atom-nextjs/` | The public npm SDK package |

## Key Files

### Entry Points & Configuration
- **`app/layout.tsx`** — Root layout; sets Montserrat font, renders `<Toaster>` for toast notifications.
- **`middleware.ts`** — Applies Upstash rate limiting (30 req/min sliding window) to all `/api/*` routes.
- **`next.config.mjs`** — Sets `Cache-Control: no-store` headers on `/`, `/app/*`, and `/api/*`.
- **`tailwind.config.ts`** — TailwindCSS config with shadcn/ui theme, typography plugin, and custom CSS variable injection. Content includes `atom-nextjs` package sources.
- **`components.json`** — shadcn/ui configuration (default style, RSC enabled, slate base color, CSS variables).

### Authentication
- **`lib/server/lucia/init.ts`** — Initializes Lucia with MongoDB adapter, configures session cookies (non-expiring, secure in production).
- **`lib/server/lucia/functions/validate-request.ts`** — Cached (`react/cache`) session validation; reads session cookie, validates via Lucia, refreshes session if needed.
- **`lib/server/encoding/encodePassword.ts`** — Hashes passwords with Argon2 + salt from env.
- **`lib/server/encoding/isPasswordValid.ts`** — Verifies passwords against Argon2 hashes.
- **`components/containers/ProtectedRoute.tsx`** — Server component that validates session and redirects to `/signin` if unauthenticated. Used as layout wrapper for `/app/*`.

### Database
- **`lib/server/mongo/init.ts`** — Connects to MongoDB, exports Mongoose model references: `UserCredentialsRef`, `UserDocumentsRef`, `ProjectsRef`, `SessionRef`.
- **`lib/server/mongo/types/userCredentials.ts`** — Schema for auth credentials (email, password_hash, custom string _id).
- **`lib/server/mongo/types/userDocuments.ts`** — Schema for user profiles (name, email, plan, embedded projects array).
- **`lib/server/mongo/types/userProjects.ts`** — Schema for projects (title, project_key, embedded posts array with full post schema).
- **`lib/server/mongo/types/userSessions.ts`** — Schema for Lucia sessions (user_id, expires_at).

### API Routes
- **`app/api/auth/signup/route.ts`** — Creates user credentials + document in a Mongoose transaction, creates Lucia session. Defines the shared `ApiResponse<T>` type.
- **`app/api/auth/signin/route.ts`** — Validates credentials, creates Lucia session.
- **`app/api/auth/signout/route.ts`** — Invalidates Lucia session, clears cookie.
- **`app/api/auth/delete/route.ts`** — Deletes user credentials, documents, projects, and sessions in a transaction. Requires password confirmation.
- **`app/api/auth/user/get/route.ts`** — Returns current user's document. Max duration: 15s.
- **`app/api/auth/user/update/route.ts`** — Updates first_name/last_name.
- **`app/api/posts/create/route.ts`** — Creates post in a project (checks plan limits on body length).
- **`app/api/posts/delete/route.ts`** — Removes post from project (owner auth).
- **`app/api/posts/update/route.ts`** — Updates post fields using `$set` with positional operator.
- **`app/api/posts/get/single/route.ts`** — Public endpoint; fetches post by Bearer project_key. Appends watermark for "single" plan.
- **`app/api/projects/create/route.ts`** — Creates project + updates user's projects array in transaction (checks plan project limit).
- **`app/api/projects/delete/route.ts`** — Deletes project + removes from user's projects array in transaction.
- **`app/api/projects/get/single/route.ts`** — Dual-mode: Bearer token returns project by key; session auth returns by project_id.
- **`app/api/projects/get/single/client/route.ts`** — Public endpoint returning sanitized client-safe project data (no sensitive fields like project_key or creator_uid on posts).

### Client-Side API Wrappers
- **`lib/client/auth/*.ts`** — axios wrappers for auth endpoints; throw on `!success`.
- **`lib/client/posts/*.ts`** — axios wrappers for post CRUD; show toast on success.
- **`lib/client/projects/*.ts`** — axios wrappers for project CRUD; show toast on success.

### SDK Package (`packages/atom-nextjs/`)
- **`src/index.tsx`** — Public exports: `Atom`, `AtomBody`, `AtomPage`, `AtomPostCard`, `AtomLoadingSkeleton`, `AtomArticleSkeleton`, `generatePostMetadata`, `getPost`, `getProject`, `generateSitemap`.
- **`src/components/Atom.tsx`** — Async server component that fetches a post by key + ID and renders full article with title, image, author, date, and MDX body.
- **`src/components/AtomBody.tsx`** — Compiles MDX content with `next-mdx-remote/rsc`, using remark-gfm and rehype-sanitize.
- **`src/components/AtomPage.tsx`** — Async server component that fetches all posts for a project and renders post cards.
- **`src/lib/client/getPost.ts`** / **`getProject.ts`** — Fetch functions using Bearer auth against the Atom API.
- **`src/lib/client/generatePostMetadata.ts`** — Generates Next.js `Metadata` for a post (title, description, keywords, author).
- **`src/lib/client/generateSitemap.ts`** — Generates sitemap entries for all posts in a project.

### Core Constants
- **`lib/contants.tsx`** — Defines: `plans` (`["single", "startup", "business"]`), `navOptions` (sidebar items), `maxInputLength` (30), `projectTitleMaxLength` (50), `baseAPIRoute` (env-dependent), `mongoDBURI`, `planDetails` (pricing tiers with limits), `npmPackage` command.

## Data Model

### TypeScript Types (`lib/types.ts`)

```typescript
type UserCredentials = {
  email: string;
  password_hash: string;
  createdAt: Date;
  updatedAt: Date;
  _id: string;               // UUID
};

type UserDocument = {
  _id: string;                // Same UUID as credentials
  first_name: string;
  last_name: string;
  email: string;
  plan: Plan;                 // "single" | "startup" | "business"
  projects: UserDocumentProjects[];  // Embedded project references
  createdAt: Date;
  updatedAt: Date;
};

type UserDocumentProjects = {
  id: string;                 // Project UUID
  title: string;
  createdAt: Date;
  updatedAt: Date;
  creator: { uid: string; email: string };
};

type Project = {
  _id: string;                // UUID
  title: string;
  posts: Post[];              // Embedded array of posts
  project_key: string;        // "atom-" + base64 random key
  creator_uid: string;        // User UUID
  createdAt: Date;
  updatedAt: Date;
};

type Post = {
  id: string;                 // UUID
  title: string;
  author: string;
  body: string;               // Markdown/MDX content
  image: string | null;
  creator_uid: string;
  keywords?: string[];
  teaser: string;
  createdAt: Date;
  updatedAt: Date;
};

type Session = {
  user_id: string;
  expires_at: Date;
};

type Plan = "single" | "startup" | "business";

type PlanDetailsPlan = {
  title: string;
  id: Plan;
  price: number | null;
  description: string;
  max_docs: number;
  max_body_length: number;
  max_projects: number;
  features: string[];
  active: boolean;
  disabled: boolean;
};
```

### MongoDB Collections
| Collection | Mongoose Model | Schema |
|---|---|---|
| `credentials` | `UserCredentialsRef` | email (unique, lowercase), password_hash, _id (string) |
| `documents` | `UserDocumentsRef` | _id, first_name, last_name, email, plan (enum), projects (embedded array) |
| `projects` | `ProjectsRef` | _id, title, posts (embedded array), project_key, creator_uid |
| `sessions` | `SessionRef` | user_id, expires_at (managed by Lucia) |

### Relationships
- **User ↔ Credentials**: 1:1 via shared `_id` (UUID).
- **User ↔ Projects**: 1:N. Projects are stored as standalone documents AND as embedded references in the user's `documents.projects` array (denormalized).
- **Project ↔ Posts**: 1:N. Posts are fully embedded within the project document.
- Writes that span collections use **Mongoose transactions** (`mongooseSession.withTransaction()`).

## API / Routes

### Auth Endpoints

| Method | Path | Body/Params | Auth | Description |
|--------|------|------------|------|-------------|
| `POST` | `/api/auth/signup` | `{ email, password, first_name, last_name }` | None | Register user, create session |
| `POST` | `/api/auth/signin` | `{ email, password }` | None | Login, create session |
| `POST` | `/api/auth/signout` | — | Session cookie | Invalidate session |
| `DELETE` | `/api/auth/delete` | `{ password }` | Session cookie | Delete account + all data |
| `GET` | `/api/auth/user/get` | — | Session cookie | Get current user document |
| `PATCH` | `/api/auth/user/update` | `{ first_name?, last_name? }` | Session cookie | Update profile |

### Posts Endpoints

| Method | Path | Body/Params | Auth | Description |
|--------|------|------------|------|-------------|
| `POST` | `/api/posts/create?project_id=` | `{ title, author, body, image?, keywords?, teaser }` | Session cookie | Create post |
| `PATCH` | `/api/posts/update?project_id=&post_id=` | `{ title?, author?, body?, teaser?, keywords?, image? }` | Session cookie | Update post |
| `DELETE` | `/api/posts/delete?project_id=&post_id=` | — | Session cookie | Delete post |
| `GET` | `/api/posts/get/single?post_id=` | — | Bearer project_key | Fetch single post (public) |

### Projects Endpoints

| Method | Path | Body/Params | Auth | Description |
|--------|------|------------|------|-------------|
| `POST` | `/api/projects/create` | `{ title }` | Session cookie | Create project |
| `DELETE` | `/api/projects/delete?project_id=` | — | Session cookie | Delete project |
| `GET` | `/api/projects/get/single?project_id=` | — | Session cookie OR Bearer key | Fetch project |
| `GET` | `/api/projects/get/single/client` | — | Bearer project_key | Fetch client-safe project data |

### Shared Response Type
All API endpoints return:
```typescript
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

### Page Routes

| Path | Auth | Description |
|------|------|-------------|
| `/` | Public | Marketing landing page |
| `/signin` | Public | Login page |
| `/signup` | Public | Registration page |
| `/pricing` | Public | Pricing plans page |
| `/blog` | Public | Blog listing (via atom-nextjs SDK) |
| `/blog/[id]` | Public | Single blog post (via atom-nextjs SDK) |
| `/app` | Protected | Dashboard — projects list |
| `/app/projects/[id]` | Protected | Project editor (post management) |
| `/app/settings` | Protected | User settings (name update, delete account) |
| `/app/settings/billing` | Protected | Billing (placeholder) |

## Dependencies

### Main Application
| Package | Purpose |
|---------|---------|
| `next` (14.1.0) | React framework (App Router) |
| `lucia` (3.1.1) | Session-based authentication |
| `@lucia-auth/adapter-mongodb` | Lucia adapter for MongoDB |
| `mongoose` (8.1.2) | MongoDB ODM |
| `argon2` (0.40.1) | Password hashing |
| `@upstash/redis` + `@upstash/ratelimit` | Redis-based rate limiting |
| `atom-nextjs` (0.3.1) | Self-consuming own SDK for the demo blog |
| `zod` (3.22.4) | Schema validation (forms, API inputs) |
| `react-hook-form` + `@hookform/resolvers` | Form state management with Zod resolver |
| `@tanstack/react-table` | Data table for projects list |
| `@tanstack/react-query` | (Available but not actively used in visible code) |
| `axios` | HTTP client for client-side API calls |
| `@uiw/react-md-editor` | Markdown editor in post forms |
| `react-hot-toast` | Toast notifications |
| `zustand` | State management (available, not visibly used) |
| `@radix-ui/*` | Headless UI primitives (via shadcn/ui) |
| `class-variance-authority` + `clsx` + `tailwind-merge` | Styling utilities (shadcn pattern) |
| `framer-motion` | Animations |
| `react-syntax-highlighter` | Code syntax highlighting on landing page |
| `uuid` | UUID generation for IDs |

### SDK Package (`atom-nextjs`)
| Package | Purpose |
|---------|---------|
| `next` | Peer dependency, server components |
| `next-mdx-remote` | MDX compilation for post body rendering |
| `remark-gfm` (3.0.0) | GitHub-flavored markdown support |
| `rehype-sanitize` | HTML sanitization in rendered MDX |
| `react-loading-skeleton` | Loading skeleton components |
| `tsdx` | Build tooling for the package |

## Build & Run

### Main Application
```bash
# Install dependencies
npm install  # or bun install

# Development
npm run dev      # Starts Next.js dev server at http://localhost:3000

# Production build
npm run build    # Builds Next.js app
npm run start    # Starts production server

# Linting
npm run lint     # Runs next lint (ESLint)
```

### SDK Package (`packages/atom-nextjs/`)
```bash
cd packages/atom-nextjs

# Development (watch mode)
npm start        # tsdx watch

# Build
npm run build    # tsdx build

# Link locally for development
npm link                        # In packages/atom-nextjs/
cd ../..                        # Back to root
npm link atom-nextjs            # Link the local package

# Test
npm test         # tsdx test --passWithNoTests
```

### Environment Variables
```bash
HASH_SALT="..."                    # Salt appended to passwords before Argon2 hashing
MONGO_DB_URI="..."                 # MongoDB connection string
ATOM_PROJECT_KEY="..."             # Project key for the demo blog
UPSTASH_REDIS_REST_URL="..."       # Upstash Redis REST URL
UPSTASH_REDIS_REST_TOKEN="..."     # Upstash Redis REST token
ENV="dev"                          # "dev" or "prod" — controls baseAPIRoute
NEXT_PUBLIC_ENV="dev"              # Public env var for client-side API base URL
```

## Patterns & Conventions

### Code Organization
- **`@/*` path alias** maps to project root (configured in `tsconfig.json`).
- **Server vs Client split**: `lib/server/` contains server-only code (never imported from `"use client"` components). `lib/client/` contains client-side API wrappers.
- **API wrappers**: Every API call from the client goes through a dedicated function in `lib/client/` that uses axios, checks `success`, throws on failure, and optionally shows a toast.

### Component Patterns
- **Server Components** are the default. Client components are marked with `"use client"` directive.
- **Container components** (`AppContainer`, `MainContainer`) provide layout structure.
- **ProtectedRoute** is an async server component used as a layout wrapper — validates session server-side and redirects if unauthenticated.
- **shadcn/ui** components in `components/ui/` follow the standard shadcn pattern with `cn()` utility.
- Forms use **react-hook-form** + **Zod** schemas + **@hookform/resolvers/zod**.

### API Patterns
- All API route handlers follow the same structure: try/catch → return `NextResponse.json<ApiResponse<T>>`.
- Error responses always include `{ success: false, message: err.message || err, response: null }`.
- Mutations that span multiple collections use **Mongoose transactions** (`mongooseSession.withTransaction()`).
- **Authorization** is checked by comparing `user.id` (from Lucia session) with `project.creator_uid`.
- Plan-based limits (max body length, max projects) are enforced in API routes by looking up `planDetails`.

### Naming Conventions
- File names: camelCase for utilities (`fetchUser.ts`), PascalCase for components (`ProjectPage.tsx`).
- Route files: always `route.ts` following Next.js convention.
- Types: PascalCase (`UserDocument`, `Post`, `Project`).
- Constants file is misspelled as `contants.tsx` (not `constants`) — this is an existing typo in the codebase.

### Error Handling
- Server: try/catch in every API route, errors returned as JSON (never thrown to the client as 500s).
- Client: API wrappers throw errors with the server's `message` field; components catch these and show `toast.error()`.
- MongoDB duplicate key errors (code 11000) are specially handled in signup to return "Email already in use."

### Authentication Flow
1. **Signup/Signin** → Creates Lucia session → Sets session cookie via `cookies().set()`.
2. **Protected pages** → `ProtectedRoute` layout calls `validateRequest()` → Redirects to `/signin` if no valid session.
3. **API routes** → Call `validateRequest()` to get `user` object → Check ownership (`user.id === resource.creator_uid`).
4. **Public SDK access** → Uses `Authorization: Bearer <project_key>` header → Looks up project by `project_key` field.

### Key Gotchas
- Posts are **embedded documents** inside the Project document, not separate collections. Updates use MongoDB positional operators (`posts.$.field`).
- User projects list is **denormalized** — stored both in the `projects` collection and embedded in `documents.projects`. Both must be updated in sync via transactions.
- `baseAPIRoute` differs between dev (`localhost:3000`) and prod (`cmsatom.netlify.app`), controlled by `NEXT_PUBLIC_ENV`.
- The SDK hardcodes its own `baseAPIRoute` to `https://cmsatom.netlify.app/api` in `packages/atom-nextjs/src/lib/constants.ts`.
