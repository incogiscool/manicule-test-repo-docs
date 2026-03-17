Now I have a comprehensive understanding of the entire codebase. Let me produce the CONTEXT.md:

# Atom CMS

## Overview
Atom is a **Content Management System (CMS) built for Next.js** that allows users to create, edit, and publish blog posts through a web dashboard, then render them in their own Next.js applications via an npm SDK package (`atom-nextjs`). The stack is **Next.js 14 (App Router)**, **TypeScript**, **MongoDB (Mongoose)**, **Lucia Auth** for session-based authentication, **Upstash Redis** for API rate limiting, **TailwindCSS** with **shadcn/ui** components, and **Zod** + **React Hook Form** for validation. The project is licensed under Apache 2.0.

## File Tree
```
context/
  randomcode.ts                         # Trivial test file (console.log)
  test-code/
    .eslintrc.json
    .gitignore
    LICENSE                             # Apache 2.0
    README.md
    bun.lock
    components.json                     # shadcn/ui config
    middleware.ts                        # Rate-limiting middleware for /api routes
    next.config.mjs
    package.json
    package-lock.json
    postcss.config.js
    tailwind.config.ts
    tsconfig.json
    app/
      favicon.ico
      globals.css                       # Tailwind + CSS variables (shadcn theme)
      layout.tsx                        # Root layout (Montserrat font, Toaster)
      page.tsx                          # Marketing landing page
      robots.ts                         # SEO robots.txt
      sitemap.ts                        # Dynamic sitemap using atom-nextjs SDK
      blog/
        page.tsx                        # Public blog list (uses AtomPage SDK component)
        [id]/
          page.tsx                      # Public blog post (uses Atom SDK component)
      pricing/
        page.tsx                        # Pricing plans page
      signin/
        page.tsx                        # Sign-in page
      signup/
        page.tsx                        # Sign-up page
      app/
        layout.tsx                      # Protected layout (redirects if unauthenticated)
        page.tsx                        # Dashboard home — projects list
        projects/
          [id]/
            page.tsx                    # Single project view (post editor)
        settings/
          page.tsx                      # User settings (name update, delete account)
          billing/
            page.tsx                    # Billing page (coming soon)
      api/
        auth/
          signup/
            route.ts                    # POST — create account
          signin/
            route.ts                    # POST — log in
          signout/
            route.ts                    # POST — log out
          delete/
            route.ts                    # DELETE — delete user & all data
          user/
            get/
              route.ts                  # GET — fetch current user document
            update/
              route.ts                  # PATCH — update user name
        posts/
          create/
            route.ts                    # POST — create post in project
          delete/
            route.ts                    # DELETE — delete post from project
          update/
            route.ts                    # PATCH — update post fields
          get/
            single/
              route.ts                  # GET — get single post (public, by project_key)
        projects/
          create/
            route.ts                    # POST — create project
          delete/
            route.ts                    # DELETE — delete project
          get/
            single/
              route.ts                  # GET — get project (by session or project_key)
              client/
                route.ts               # GET — get project for public client (sanitized)
    components/
      cards/
        PricingPlanCard.tsx
      containers/
        AppContainer.tsx               # Dashboard layout with sidebar
        MainContainer.tsx              # Public page layout with navbar + footer
        ProtectedRoute.tsx             # Server component auth guard
      forms/
        LoginForm.tsx                  # Client-side login form
        SignupForm.tsx                 # Client-side signup form
      misc/
        NpmPackageComponent.tsx        # Copy-to-clipboard npm install command
        tracing-beam.tsx               # Aceternity UI animated beam
      modals/
        CreatePostModal.tsx            # Dialog to create a new post
        DeleteUserModal.tsx            # Dialog to confirm user deletion
      nav/
        Navbar.tsx                     # Public-facing top navbar
      pages/
        projects/
          ProjectComponent.tsx         # Project view with sidebar + post editor
          ProjectFormComponent.tsx     # Post editing form (zod-validated)
          ProjectPage.tsx              # Projects list with create dialog
        settings/
          SettingsForm.tsx             # User settings form
      sidebars/
        AppSidebarNav.tsx              # Dashboard sidebar navigation
        ProjectComponentSidebar.tsx    # Post list sidebar within a project
      tables/
        UserDocumentProjects/
          columns.tsx                  # TanStack Table column definitions
          table.tsx                    # TanStack Table component
      ui/                              # shadcn/ui primitives
        alert-dialog.tsx
        button.tsx
        carousel.tsx
        dialog.tsx
        dropdown-menu.tsx
        form.tsx
        input.tsx
        label.tsx
        markdown-editor.tsx            # @uiw/react-md-editor wrapper
        popover.tsx
        sticky-scroll-reveal.tsx
        table.tsx
        textarea.tsx
    lib/
      contants.tsx                     # Plans, nav options, API base URL, limits
      types.ts                         # Core TypeScript types
      utils.ts                         # cn() helper (clsx + tailwind-merge)
      utils/
        validateEmail.ts               # Email regex validator
      client/
        auth/
          loginUser.ts
          signupUser.ts
          signoutUser.ts
          deleteUser.ts
          updateUser.ts
        posts/
          createPost.ts
          deletePost.ts
          updatePost.ts
        projects/
          createProject.ts
          deleteProject.ts
      server/
        encoding/
          encodePassword.ts            # Argon2 password hashing
          isPasswordValid.ts           # Argon2 password verification
        functions/
          user/
            fetchUser.ts              # Server-side user fetch via internal API
          projects/
            getProject.ts            # Server-side project fetch via internal API
        lucia/
          init.ts                     # Lucia auth initialization with MongoDB adapter
          functions/
            validate-request.ts       # Cached session validation
        mongo/
          init.ts                     # MongoDB connection + Mongoose model exports
          types/
            userCredentials.ts        # Mongoose schema: credentials
            userDocuments.ts          # Mongoose schema: user documents
            userProjects.ts           # Mongoose schema: projects + posts
            userSessions.ts           # Mongoose schema: sessions
        redis/
          init.ts                     # Upstash Redis + rate limiter init
        utils/
          generateProjectKey.ts       # Generates "atom-" prefixed random key
          validateProjectKey.ts       # Validates project key against DB
          validateRequestFetchUser.ts # Combines session validation + user fetch
    bruno/
      bruno.json                      # Bruno API collection config
      environments/
        Development.bru
      Routes/
        Create Post.bru
        Create Project.bru
        Delete Post.bru
        Delete Project.bru
        Get Post.bru
        Get Project.bru
        Get User.bru
    public/
      atom-black.svg
      next.svg
      vercel.svg
    packages/
      atom-nextjs/                    # Published npm SDK package
        .gitignore
        .github/workflows/
          main.yml
          size.yml
        LICENSE
        README.md                     # SDK usage documentation
        package.json
        package-lock.json
        tsconfig.json
        yarn.lock
        src/
          index.tsx                   # Package entry — exports all components + utils
          components/
            Atom.tsx                  # Single post renderer (async server component)
            AtomArticleSkeleton.tsx   # Loading skeleton for article page
            AtomBody.tsx             # MDX body compiler (remark-gfm, rehype-sanitize)
            AtomLoadingSkeleton.tsx   # Loading skeleton for post list page
            AtomPage.tsx             # Post list renderer (async server component)
            AtomPostCard.tsx         # Individual post card component
          lib/
            constants.ts             # SDK base API URL
            types.ts                 # SDK type definitions
            client/
              generatePostMetadata.ts # Next.js metadata generator for posts
              generateSitemap.ts      # Sitemap generator for blog posts
              getPost.ts             # Fetches single post via project key
              getProject.ts          # Fetches project (client-safe) via project key
```

## Architecture

### Overview
The application follows a **monorepo-like structure** with two main parts:

1. **Main Next.js App** (`context/test-code/`) — The CMS dashboard and API server
2. **SDK Package** (`context/test-code/packages/atom-nextjs/`) — An npm package consumers install to render blog content

### Data Flow

```
┌──────────────────┐     ┌───────────────────────┐     ┌──────────────┐
│  Consumer Site   │────▶│  Next.js API Routes    │────▶│   MongoDB    │
│  (atom-nextjs)   │     │  (/api/*)              │     │  (Mongoose)  │
└──────────────────┘     └───────────────────────┘     └──────────────┘
                              │         ▲
                              │         │
                              ▼         │
                         ┌────────────────┐
                         │  Lucia Auth    │
                         │  (Sessions)    │
                         └────────────────┘
                              │
                              ▼
                         ┌────────────────┐
                         │ Upstash Redis  │
                         │ (Rate Limiting)│
                         └────────────────┘
```

**Authentication flow**: Users sign up/in via forms → API routes create Lucia sessions stored in MongoDB → session cookie (`auth_session`) is set → subsequent requests validate via `validateRequest()`.

**Consumer flow**: External Next.js apps install `atom-nextjs`, provide their `ATOM_PROJECT_KEY`, and the SDK's server components fetch posts from the Atom API using Bearer token auth.

**Dashboard flow**: Authenticated users create projects → get a `project_key` → create/edit/delete posts within projects → posts are stored as nested arrays within project documents in MongoDB.

### Directory Responsibilities

| Directory | Purpose |
|---|---|
| `app/api/` | REST API routes (auth, posts, projects) |
| `app/app/` | Authenticated dashboard pages (protected by `ProtectedRoute`) |
| `app/blog/`, `app/pricing/`, `app/signin/`, `app/signup/` | Public-facing marketing & auth pages |
| `components/` | Reusable React components (forms, modals, sidebars, tables, UI primitives) |
| `lib/client/` | Client-side API wrapper functions (called from components) |
| `lib/server/` | Server-only utilities (auth, DB, encoding, Redis) |
| `packages/atom-nextjs/` | Publishable npm SDK for rendering blog content |

## Key Files

### Entry Points & Configuration
- **`app/layout.tsx`** — Root layout with Montserrat font and `react-hot-toast` Toaster.
- **`middleware.ts`** — Intercepts `/api/*` requests to apply Upstash Redis rate limiting (30 requests per minute per IP).
- **`next.config.mjs`** — Disables caching (`no-store`) for `/`, `/app/*`, and `/api/*` routes.
- **`tailwind.config.ts`** — Includes shadcn/ui theming, `@tailwindcss/typography` plugin, and scans `atom-nextjs` package sources.
- **`components.json`** — shadcn/ui configuration (default style, RSC enabled, `@/` aliases).

### Authentication
- **`lib/server/lucia/init.ts`** — Initializes Lucia v3 with MongoDB adapter, configures session cookies.
- **`lib/server/lucia/functions/validate-request.ts`** — Cached (React `cache()`) session validation that reads cookies, validates with Lucia, and auto-refreshes sessions.
- **`lib/server/encoding/encodePassword.ts`** — Hashes passwords with Argon2 + environment salt.
- **`lib/server/encoding/isPasswordValid.ts`** — Verifies passwords against Argon2 hashes.
- **`components/containers/ProtectedRoute.tsx`** — Server component that validates sessions and redirects to `/signin` if unauthenticated.

### Database
- **`lib/server/mongo/init.ts`** — Mongoose connection + exports four model references: `UserCredentialsRef`, `UserDocumentsRef`, `ProjectsRef`, `SessionRef`.
- **`lib/server/mongo/types/userCredentials.ts`** — Schema for login credentials (email, password_hash, _id).
- **`lib/server/mongo/types/userDocuments.ts`** — Schema for user profiles (name, email, plan, projects array).
- **`lib/server/mongo/types/userProjects.ts`** — Schema for projects (title, posts array, project_key, creator_uid) and nested post schema.
- **`lib/server/mongo/types/userSessions.ts`** — Schema for auth sessions (user_id, expires_at).

### API Routes
- **`app/api/auth/signup/route.ts`** — Creates user credentials + user document in a transaction, creates Lucia session. Exports the `ApiResponse<T>` type used everywhere.
- **`app/api/auth/signin/route.ts`** — Validates email/password, creates Lucia session.
- **`app/api/auth/signout/route.ts`** — Invalidates session, clears cookie.
- **`app/api/auth/delete/route.ts`** — Deletes all user data (credentials, documents, projects, sessions) in a transaction.
- **`app/api/posts/create/route.ts`** — Creates a post within a project, enforces plan limits on body length.
- **`app/api/posts/update/route.ts`** — Partially updates post fields using MongoDB positional operator.
- **`app/api/posts/delete/route.ts`** — Removes a post from a project's posts array.
- **`app/api/posts/get/single/route.ts`** — Public endpoint for fetching a single post via Bearer project key. Appends watermark for free plan users.
- **`app/api/projects/create/route.ts`** — Creates a project with a generated key, enforces plan project limits, uses transactions.
- **`app/api/projects/delete/route.ts`** — Deletes project document and removes from user's projects array in a transaction.
- **`app/api/projects/get/single/route.ts`** — Fetches project by session (dashboard) or by Bearer key (SDK).
- **`app/api/projects/get/single/client/route.ts`** — Public client endpoint that returns sanitized project data (strips sensitive fields like `project_key`, `creator_uid`).

### Client API Wrappers
- **`lib/client/auth/*.ts`** — Axios wrappers for auth endpoints (`loginUser`, `signupUser`, `signoutUser`, `deleteUser`, `updateUser`).
- **`lib/client/posts/*.ts`** — Axios wrappers for post CRUD (`createPost`, `deletePost`, `updatePost`).
- **`lib/client/projects/*.ts`** — Axios wrappers for project CRUD (`createProject`, `deleteProject`).

### SDK Package (`packages/atom-nextjs/`)
- **`src/index.tsx`** — Package entry; exports: `Atom`, `AtomBody`, `AtomPage`, `AtomPostCard`, `AtomLoadingSkeleton`, `AtomArticleSkeleton`, `generatePostMetadata`, `getPost`, `getProject`, `generateSitemap`.
- **`src/components/Atom.tsx`** — Async server component that fetches and renders a single blog post with MDX body.
- **`src/components/AtomPage.tsx`** — Async server component that fetches and renders a grid of post cards.
- **`src/components/AtomBody.tsx`** — Compiles MDX content using `next-mdx-remote/rsc` with `remark-gfm` and `rehype-sanitize`.
- **`src/lib/client/getPost.ts`** / **`getProject.ts`** — Fetch functions using Bearer token auth against the Atom API.

## Data Model

### Core Types (`lib/types.ts`)

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
  projects: UserDocumentProjects[];
  createdAt: Date;
  updatedAt: Date;
};

type UserDocumentProjects = {
  id: string;
  title: string;
  createdAt: Date;
  updatedAt: Date;
  creator: { uid: string; email: string };
};

type Project = {
  _id: string;                // UUID
  title: string;
  posts: Post[];              // Embedded array
  project_key: string;        // "atom-" + base64 random string
  creator_uid: string;
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
```

### MongoDB Collections
| Collection | Mongoose Model | Schema File |
|---|---|---|
| `credentials` | `UserCredentialsRef` | `lib/server/mongo/types/userCredentials.ts` |
| `documents` | `UserDocumentsRef` | `lib/server/mongo/types/userDocuments.ts` |
| `projects` | `ProjectsRef` | `lib/server/mongo/types/userProjects.ts` |
| `session` | `SessionRef` | `lib/server/mongo/types/userSessions.ts` |

### Relationships
- **UserCredentials ↔ UserDocument**: Linked by `_id` (same UUID).
- **UserDocument → Projects**: User has embedded `projects[]` array (denormalized summary), linked to full `Project` documents by `id`.
- **Project → Posts**: Posts are embedded as a subdocument array within the Project document.
- **Session → User**: `user_id` references `UserCredentials._id`.

### Plan Limits
| Plan | Price | Max Projects | Max Posts | Max Body Length |
|---|---|---|---|---|
| single | Free | 2 | 100 | 10,000 chars |
| startup | $3.99/mo | 3 | 1,000 | 100,000 chars |
| business | $11.99/mo | 5 | 2,500 | 500,000 chars |

*Note: startup and business plans are `disabled: true` (coming soon).*

## API / Routes

### Auth Routes
| Method | Path | Auth | Description |
|---|---|---|---|
| `POST` | `/api/auth/signup` | None | Create account. Body: `{ email, password, first_name, last_name }`. Returns `ApiResponse<UserDocument>`. |
| `POST` | `/api/auth/signin` | None | Log in. Body: `{ email, password }`. Returns `ApiResponse<UserDocument>`. |
| `POST` | `/api/auth/signout` | Session cookie | Invalidate session. Returns `ApiResponse`. |
| `DELETE` | `/api/auth/delete` | Session cookie | Delete user and all data. Body: `{ password }`. Returns `ApiResponse`. |
| `GET` | `/api/auth/user/get` | Session cookie | Fetch current user document. Returns `ApiResponse<UserDocument>`. |
| `PATCH` | `/api/auth/user/update` | Session cookie | Update user name. Body: `{ first_name?, last_name? }`. Returns `ApiResponse`. |

### Project Routes
| Method | Path | Auth | Description |
|---|---|---|---|
| `POST` | `/api/projects/create` | Session cookie | Create project. Body: `{ title }`. Returns `ApiResponse<Project>`. |
| `DELETE` | `/api/projects/delete?project_id=X` | Session cookie | Delete project. Returns `ApiResponse`. |
| `GET` | `/api/projects/get/single?project_id=X` | Session cookie OR `Authorization: Bearer <key>` | Fetch project (full data). Returns `ApiResponse<Project>`. |
| `GET` | `/api/projects/get/single/client` | `Authorization: Bearer <project_key>` | Fetch project (sanitized for public). Returns `ApiResponse<ClientProject>`. |

### Post Routes
| Method | Path | Auth | Description |
|---|---|---|---|
| `POST` | `/api/posts/create?project_id=X` | Session cookie | Create post. Body: `{ title, author, body, image?, keywords?, teaser }`. Returns `ApiResponse<Post>`. |
| `DELETE` | `/api/posts/delete?project_id=X&post_id=Y` | Session cookie | Delete post. Returns `ApiResponse`. |
| `PATCH` | `/api/posts/update?project_id=X&post_id=Y` | Session cookie | Update post fields. Body: `{ title?, author?, body?, teaser?, keywords?, image? }`. Returns `ApiResponse`. |
| `GET` | `/api/posts/get/single?post_id=X` | `Authorization: Bearer <project_key>` | Fetch single post (public). Appends watermark for free plan. Returns `ApiResponse<Post>`. |

### Standard API Response Shape
```typescript
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

### Page Routes
| Path | Type | Description |
|---|---|---|
| `/` | Public | Marketing landing page |
| `/signin` | Public | Login form |
| `/signup` | Public | Registration form |
| `/pricing` | Public | Pricing plans display |
| `/blog` | Public | Blog post listing (via `atom-nextjs` SDK) |
| `/blog/[id]` | Public | Single blog post (via `atom-nextjs` SDK) |
| `/app` | Protected | Dashboard — projects list |
| `/app/projects/[id]` | Protected | Project view — post editor |
| `/app/settings` | Protected | User settings |
| `/app/settings/billing` | Protected | Billing (coming soon) |

## Dependencies

### Main App — Key Dependencies
| Package | Purpose |
|---|---|
| `next` 14.1.0 | React framework (App Router) |
| `react` ^18 | UI library |
| `mongoose` ^8.1.2 | MongoDB ODM |
| `lucia` ^3.1.1 | Session-based authentication |
| `@lucia-auth/adapter-mongodb` ^1.0.2 | Lucia MongoDB adapter |
| `argon2` ^0.40.1 | Password hashing |
| `@upstash/redis` + `@upstash/ratelimit` | Redis-based API rate limiting |
| `zod` ^3.22.4 | Schema validation |
| `react-hook-form` + `@hookform/resolvers` | Form state management with Zod resolver |
| `@tanstack/react-table` ^8.13.2 | Data table for projects listing |
| `@tanstack/react-query` ^5.27.5 | Data fetching (installed but minimally used) |
| `axios` ^1.6.7 | HTTP client for client-side and server-side API calls |
| `zustand` ^4.5.0 | State management (installed, not actively used in visible code) |
| `atom-nextjs` ^0.3.1 | The SDK package (self-dependency for blog rendering) |
| `@uiw/react-md-editor` ^4.0.4 | Markdown editor for post body |
| `react-hot-toast` ^2.4.1 | Toast notifications |
| `react-syntax-highlighter` ^15.5.0 | Code syntax highlighting on landing page |
| `react-markdown` ^9.0.1, `next-mdx-remote` ^4.4.1 | Markdown rendering |
| `framer-motion` ^11.0.15 | Animations |
| `uuid` ^9.0.1 | UUID generation for IDs |
| Radix UI primitives | Dialog, dropdown, alert-dialog, popover, label (via shadcn/ui) |
| `tailwindcss` + `tailwindcss-animate` + `@tailwindcss/typography` | Styling |

### SDK Package — Key Dependencies
| Package | Purpose |
|---|---|
| `next-mdx-remote` ^4.4.1 | Server-side MDX compilation |
| `remark-gfm` 3.0.0 | GitHub-flavored markdown support |
| `rehype-sanitize` ^6.0.0 | HTML sanitization |
| `react-loading-skeleton` ^3.5.0 | Loading skeleton components |
| `tsdx` ^0.14.1 | Build toolchain (dev) |

## Build & Run

### Prerequisites
- Node.js ≥ 10 (SDK), modern Node for the main app
- MongoDB instance
- Upstash Redis instance

### Environment Variables
```bash
HASH_SALT="..."                    # Salt appended to passwords before Argon2 hashing
MONGO_DB_URI="..."                 # MongoDB connection string
ATOM_PROJECT_KEY="..."             # Project key for the app's own blog
UPSTASH_REDIS_REST_URL="..."       # Upstash Redis REST URL
UPSTASH_REDIS_REST_TOKEN="..."     # Upstash Redis REST token
ENV="dev"                          # "dev" or "prod" — determines API base URL
```

### Scripts (Main App)
```bash
npm run dev        # Start Next.js dev server
npm run build      # Production build
npm run start      # Start production server
npm run lint       # ESLint
```

### Scripts (SDK Package — `packages/atom-nextjs/`)
```bash
npm run start      # tsdx watch (development)
npm run build      # tsdx build (production)
npm run test       # tsdx test
npm run lint       # tsdx lint
npm run prepare    # tsdx build (pre-publish)
npm link           # Link locally for development
```

### Local SDK Development
```bash
cd packages/atom-nextjs && npm link
cd ../.. && npm link atom-nextjs
```

## Patterns & Conventions

### API Response Pattern
All API routes return `ApiResponse<T>` with consistent shape: `{ success: boolean, message: string | null, response: T }`. Error handling uses try/catch with `NextResponse.json()`.

### Authentication Pattern
- Server components/routes call `validateRequest()` (cached) which reads cookies and validates via Lucia.
- `ProtectedRoute` server component wraps `/app/*` pages as a layout-level guard.
- API routes manually call `validateRequest()` and check `if (!user) throw new Error("Invalid session...")`.
- Public SDK endpoints use `Authorization: Bearer <project_key>` instead of session cookies.

### Database Transactions
Multi-collection writes (signup, delete user, create/delete project) use Mongoose sessions with `withTransaction()` for atomicity.

### Form Validation
- Client-side: Zod schemas + `react-hook-form` with `zodResolver`.
- Server-side: Manual validation in API route handlers (checking lengths, existence, etc.).

### Component Patterns
- **Server Components** (default): Used for pages, layouts, Navbar, containers. Can be `async` for data fetching.
- **Client Components** (`"use client"`): Used for interactive forms, sidebars, modals, tables. Marked explicitly.
- **shadcn/ui**: UI primitives in `components/ui/` — generated, not hand-written. Use `cn()` utility for class merging.

### Naming Conventions
- Files: PascalCase for components (`ProjectPage.tsx`), camelCase for utilities (`fetchUser.ts`).
- Routes: lowercase with folder-based REST structure (`/api/posts/create/route.ts`).
- Types: PascalCase (`UserDocument`, `ApiResponse`).
- Path aliases: `@/*` maps to project root.

### State Management
- Server state flows via `fetchUser()` and `getProject()` (internal HTTP calls from server components to API routes, forwarding cookies).
- Client state uses `useState` in individual components. Toast notifications via `react-hot-toast`.

### Error Handling
- API routes: try/catch blocks, errors returned as `{ success: false, message: err.message }`. MongoDB duplicate key errors (code 11000) have special handling in signup.
- Client wrappers: throw errors when `!data.success`, caught by components which display via `toast.error()`.

### Caching
- `no-store` Cache-Control headers set in `next.config.mjs` for dynamic routes.
- Many pages call `cookies()` to explicitly opt out of Next.js static caching.
- `validateRequest` is wrapped in React `cache()` to deduplicate within a single request.

### Rate Limiting
- Applied in middleware to all `/api/*` routes.
- Upstash sliding window: 30 requests per minute per IP.

### File Organization Notes
- Constants file is named `contants.tsx` (typo in original, not `constants`).
- The `lib/client/` directory contains browser-safe API wrappers using axios.
- The `lib/server/` directory contains Node.js-only code (crypto, DB, auth).
