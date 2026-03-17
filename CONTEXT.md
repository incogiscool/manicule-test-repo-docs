Now I have a comprehensive understanding of the entire codebase. Let me produce the CONTEXT.md.

# Atom CMS

## Overview
Atom is a Content Management System (CMS) built for Next.js projects. It allows users to create, edit, and publish blog posts and articles through a web dashboard, then integrate those posts into their own Next.js sites via the `atom-nextjs` SDK package. The tech stack is **Next.js 14 (App Router)**, **TypeScript**, **MongoDB** (via Mongoose), **Lucia** for session-based authentication, **Argon2** for password hashing, **Upstash Redis** for API rate limiting, **TailwindCSS** with **shadcn/ui** components, **React Hook Form + Zod** for form validation, and **Axios** for client-side HTTP calls. The project is licensed under Apache 2.0.

## File Tree
```
context/
├── randomcode.ts                              # Trivial test file (console.log)
└── test-code/
    ├── .eslintrc.json
    ├── .gitignore
    ├── LICENSE                                # Apache 2.0
    ├── README.md
    ├── bun.lock
    ├── components.json                        # shadcn/ui config
    ├── middleware.ts                           # Next.js middleware (rate limiting)
    ├── next.config.mjs
    ├── package-lock.json
    ├── package.json
    ├── postcss.config.js
    ├── tailwind.config.ts
    ├── tsconfig.json
    ├── public/
    │   ├── atom-black.svg
    │   ├── next.svg
    │   └── vercel.svg
    ├── app/
    │   ├── favicon.ico
    │   ├── globals.css
    │   ├── layout.tsx                         # Root layout (Montserrat font, Toaster)
    │   ├── page.tsx                           # Landing/marketing page
    │   ├── robots.ts                          # SEO robots.txt
    │   ├── sitemap.ts                         # Dynamic sitemap generation
    │   ├── signin/
    │   │   └── page.tsx                       # Login page
    │   ├── signup/
    │   │   └── page.tsx                       # Signup page
    │   ├── pricing/
    │   │   └── page.tsx                       # Pricing plans page
    │   ├── blog/
    │   │   ├── page.tsx                       # Blog listing (uses atom-nextjs SDK)
    │   │   └── [id]/
    │   │       └── page.tsx                   # Individual blog post (uses atom-nextjs SDK)
    │   ├── app/
    │   │   ├── layout.tsx                     # Protected route wrapper
    │   │   ├── page.tsx                       # Dashboard – projects list
    │   │   ├── projects/
    │   │   │   └── [id]/
    │   │   │       └── page.tsx               # Project detail – post editor
    │   │   └── settings/
    │   │       ├── page.tsx                   # User settings
    │   │       └── billing/
    │   │           └── page.tsx               # Billing (coming soon)
    │   └── api/
    │       ├── auth/
    │       │   ├── signup/route.ts            # POST – user registration
    │       │   ├── signin/route.ts            # POST – user login
    │       │   ├── signout/route.ts           # POST – user logout
    │       │   ├── delete/route.ts            # DELETE – delete user account
    │       │   └── user/
    │       │       ├── get/route.ts           # GET – fetch current user
    │       │       └── update/route.ts        # PATCH – update user profile
    │       ├── posts/
    │       │   ├── create/route.ts            # POST – create post in project
    │       │   ├── delete/route.ts            # DELETE – delete post from project
    │       │   ├── update/route.ts            # PATCH – update post in project
    │       │   └── get/
    │       │       └── single/route.ts        # GET – get single post (public, via Bearer key)
    │       └── projects/
    │           ├── create/route.ts            # POST – create project
    │           ├── delete/route.ts            # DELETE – delete project
    │           └── get/
    │               └── single/
    │                   ├── route.ts           # GET – get project (auth or Bearer key)
    │                   └── client/route.ts    # GET – get project (public, sanitized for client)
    ├── components/
    │   ├── cards/
    │   │   └── PricingPlanCard.tsx
    │   ├── containers/
    │   │   ├── AppContainer.tsx               # Dashboard layout with sidebar
    │   │   ├── MainContainer.tsx              # Public page layout with navbar/footer
    │   │   └── ProtectedRoute.tsx             # Server component auth guard
    │   ├── forms/
    │   │   ├── LoginForm.tsx
    │   │   └── SignupForm.tsx
    │   ├── misc/
    │   │   ├── NpmPackageComponent.tsx
    │   │   └── tracing-beam.tsx               # Aceternity UI animation component
    │   ├── modals/
    │   │   ├── CreatePostModal.tsx
    │   │   └── DeleteUserModal.tsx
    │   ├── nav/
    │   │   └── Navbar.tsx                     # Public navbar (server component)
    │   ├── pages/
    │   │   ├── projects/
    │   │   │   ├── ProjectComponent.tsx       # Project detail view with sidebar + editor
    │   │   │   ├── ProjectFormComponent.tsx   # Post editing form (markdown editor)
    │   │   │   └── ProjectPage.tsx            # Projects dashboard page
    │   │   └── settings/
    │   │       └── SettingsForm.tsx
    │   ├── sidebars/
    │   │   ├── AppSidebarNav.tsx              # Dashboard sidebar navigation
    │   │   └── ProjectComponentSidebar.tsx    # Project post list sidebar
    │   ├── tables/
    │   │   └── UserDocumentProjects/
    │   │       ├── columns.tsx                # TanStack table column definitions
    │   │       └── table.tsx                  # TanStack table component
    │   └── ui/                                # shadcn/ui primitives
    │       ├── alert-dialog.tsx
    │       ├── button.tsx
    │       ├── carousel.tsx
    │       ├── dialog.tsx
    │       ├── dropdown-menu.tsx
    │       ├── form.tsx
    │       ├── input.tsx
    │       ├── label.tsx
    │       ├── markdown-editor.tsx            # @uiw/react-md-editor wrapper
    │       ├── popover.tsx
    │       ├── sticky-scroll-reveal.tsx
    │       ├── table.tsx
    │       └── textarea.tsx
    ├── lib/
    │   ├── contants.tsx                       # App constants (plans, nav options, base API URL)
    │   ├── types.ts                           # Core TypeScript types
    │   ├── utils.ts                           # Tailwind cn() utility
    │   ├── utils/
    │   │   └── validateEmail.ts
    │   ├── client/                            # Client-side API call wrappers
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
    │   └── server/                            # Server-only utilities
    │       ├── encoding/
    │       │   ├── encodePassword.ts          # Argon2 hash
    │       │   └── isPasswordValid.ts         # Argon2 verify
    │       ├── functions/
    │       │   ├── projects/
    │       │   │   └── getProject.ts          # Server-side project fetch
    │       │   └── user/
    │       │       └── fetchUser.ts           # Server-side user fetch
    │       ├── lucia/
    │       │   ├── init.ts                    # Lucia auth initialization
    │       │   └── functions/
    │       │       └── validate-request.ts    # Cached session validation
    │       ├── mongo/
    │       │   ├── init.ts                    # Mongoose connection & model refs
    │       │   └── types/
    │       │       ├── userCredentials.ts     # Mongoose schema
    │       │       ├── userDocuments.ts       # Mongoose schema
    │       │       ├── userProjects.ts        # Mongoose schema (posts + projects)
    │       │       └── userSessions.ts        # Mongoose schema
    │       ├── redis/
    │       │   └── init.ts                    # Upstash Redis + Ratelimit setup
    │       └── utils/
    │           ├── generateProjectKey.ts      # Random base64 key generation
    │           ├── validateProjectKey.ts
    │           └── validateRequestFetchUser.ts
    ├── bruno/                                 # Bruno API testing collection
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
        └── atom-nextjs/                       # Published NPM SDK package
            ├── .github/workflows/
            │   ├── main.yml
            │   └── size.yml
            ├── .gitignore
            ├── LICENSE
            ├── README.md                      # SDK integration documentation
            ├── package.json
            ├── package-lock.json
            ├── tsconfig.json
            ├── yarn.lock
            └── src/
                ├── index.tsx                  # Package entry – re-exports all
                ├── components/
                │   ├── Atom.tsx               # Single post renderer (SSR)
                │   ├── AtomArticleSkeleton.tsx
                │   ├── AtomBody.tsx           # MDX body renderer
                │   ├── AtomLoadingSkeleton.tsx
                │   ├── AtomPage.tsx           # Blog listing page (SSR)
                │   └── AtomPostCard.tsx       # Post card component
                └── lib/
                    ├── constants.ts           # Base API route
                    ├── types.ts               # SDK types
                    └── client/
                        ├── generatePostMetadata.ts
                        ├── generateSitemap.ts
                        ├── getPost.ts
                        └── getProject.ts
```

## Architecture

### High-level
Atom is a **monorepo-style** Next.js 14 application with an embedded NPM package (`packages/atom-nextjs`). The main app serves dual purposes:

1. **CMS Dashboard** (`/app/*` routes): An authenticated web UI where users manage projects and author blog posts with a Markdown editor.
2. **Public API** (`/api/*` routes): REST endpoints consumed both by the dashboard and by the `atom-nextjs` SDK from external sites.
3. **Marketing Site** (`/`, `/pricing`, `/blog`): A public-facing landing page, pricing page, and a demo blog that dogfoods the SDK.

### Data Flow
1. **Authentication**: Users sign up/in → password hashed with Argon2 → credentials stored in MongoDB `credentials` collection → session created via Lucia → session cookie set. All `/app/*` routes are wrapped in `ProtectedRoute` (server component that validates session).
2. **Project Management**: Authenticated users create projects → each project gets a unique `project_key` → project stored in MongoDB `projects` collection with embedded `posts` array. A reference is also stored in the user's `documents` record.
3. **Post Management**: Posts are embedded documents within a project. Created/updated/deleted via API routes that verify session ownership.
4. **SDK Consumption**: External Next.js sites use `atom-nextjs` SDK components (`<Atom>`, `<AtomPage>`) which fetch posts from the public API using Bearer token `project_key` auth.
5. **Rate Limiting**: All `/api/*` requests pass through Next.js middleware that uses Upstash Redis sliding window rate limiter (30 requests/minute per IP).

### Key Architectural Decisions
- Posts are **embedded in project documents** (not separate collections) — fetched via MongoDB `$push`/`$pull` array operations.
- **Mongoose transactions** are used for multi-collection writes (signup creates both credentials + document; project create updates both projects + user document).
- Server-side data fetching in pages uses internal HTTP calls via Axios (e.g., `fetchUser` calls `/api/auth/user/get` with forwarded cookies) rather than direct DB access.
- The `atom-nextjs` SDK is a **server-rendered** React component library (uses `next-mdx-remote` for MDX compilation on the server).

## Key Files

### Entry Points & Config
- **`app/layout.tsx`** — Root layout: Montserrat font, `react-hot-toast` Toaster provider.
- **`middleware.ts`** — Rate limits all `/api/*` routes using Upstash Redis (30 req/min per IP).
- **`next.config.mjs`** — Sets `Cache-Control: no-store` headers on `/`, `/app/*`, and `/api/*`.
- **`tailwind.config.ts`** — Includes shadcn/ui theme, typography plugin, and atom-nextjs package in content paths.
- **`components.json`** — shadcn/ui configuration (default style, RSC enabled, `@/` aliases).

### Authentication
- **`lib/server/lucia/init.ts`** — Initializes Lucia auth with MongoDB adapter, configures session cookies.
- **`lib/server/lucia/functions/validate-request.ts`** — Cached session validator; reads session cookie, validates with Lucia, refreshes if needed.
- **`lib/server/encoding/encodePassword.ts`** — Hashes passwords with Argon2 + env salt.
- **`lib/server/encoding/isPasswordValid.ts`** — Verifies passwords against Argon2 hashes.
- **`components/containers/ProtectedRoute.tsx`** — Server component guard that redirects to `/signin` if not authenticated.

### Database
- **`lib/server/mongo/init.ts`** — Mongoose connection, exports model references: `UserCredentialsRef`, `UserDocumentsRef`, `ProjectsRef`, `SessionRef`.
- **`lib/server/mongo/types/userCredentials.ts`** — Schema for `credentials` collection (email, password_hash, _id).
- **`lib/server/mongo/types/userDocuments.ts`** — Schema for `documents` collection (user profile, embedded projects array, plan).
- **`lib/server/mongo/types/userProjects.ts`** — Schema for `projects` collection (title, embedded posts array, project_key, creator_uid).
- **`lib/server/mongo/types/userSessions.ts`** — Schema for `session` collection (user_id, expires_at).

### API Routes
- **`app/api/auth/signup/route.ts`** — Defines the `ApiResponse<T>` generic type used across all API routes. Handles user registration with transaction.
- **`app/api/auth/signin/route.ts`** — Handles login, verifies password, creates session.
- **`app/api/auth/signout/route.ts`** — Invalidates session, clears cookie.
- **`app/api/auth/delete/route.ts`** — Deletes user and all related data (credentials, projects, documents, sessions) in a transaction.
- **`app/api/auth/user/get/route.ts`** — Returns current user's document.
- **`app/api/auth/user/update/route.ts`** — Updates user's first/last name.
- **`app/api/projects/create/route.ts`** — Creates project with generated key, enforces plan limits.
- **`app/api/projects/delete/route.ts`** — Deletes project and removes reference from user document.
- **`app/api/projects/get/single/route.ts`** — Gets project by ID (auth) or by project_key (Bearer token).
- **`app/api/projects/get/single/client/route.ts`** — Public endpoint for SDK; returns sanitized project data (no full post bodies, no project_key).
- **`app/api/posts/create/route.ts`** — Creates post (pushes to project's posts array), enforces body length limits.
- **`app/api/posts/delete/route.ts`** — Removes post from project's posts array.
- **`app/api/posts/update/route.ts`** — Partially updates post fields using MongoDB positional `$set`.
- **`app/api/posts/get/single/route.ts`** — Public endpoint; returns single post by Bearer key auth; appends watermark for "single" plan users.

### Client-Side API Wrappers
- **`lib/client/auth/*`** — Functions wrapping Axios calls: `signupUser`, `loginUser`, `signoutUser`, `updateUser`, `deleteUser`.
- **`lib/client/posts/*`** — Functions: `createPost`, `updatePost`, `deletePost`.
- **`lib/client/projects/*`** — Functions: `createProject`, `deleteProject`.

### SDK Package (`packages/atom-nextjs`)
- **`src/index.tsx`** — Re-exports all public components and utilities.
- **`src/components/Atom.tsx`** — Server component that fetches a single post and renders it with MDX.
- **`src/components/AtomPage.tsx`** — Server component that fetches all posts in a project and renders cards.
- **`src/components/AtomBody.tsx`** — MDX renderer using `next-mdx-remote/rsc` with `remark-gfm` and `rehype-sanitize`.
- **`src/components/AtomPostCard.tsx`** — Linked card component for blog listings.
- **`src/lib/client/getPost.ts`** — Fetches single post via Bearer auth.
- **`src/lib/client/getProject.ts`** — Fetches project via Bearer auth.
- **`src/lib/client/generatePostMetadata.ts`** — Generates Next.js `Metadata` from a post.
- **`src/lib/client/generateSitemap.ts`** — Generates sitemap entries from project posts.

### Constants & Types
- **`lib/contants.tsx`** — Plans (`single`, `startup`, `business`), nav options, `baseAPIRoute`, plan details with limits, `maxInputLength`, `projectTitleMaxLength`.
- **`lib/types.ts`** — Core domain types: `UserCredentials`, `Post`, `Plan`, `UserDocumentProjects`, `Project`, `UserDocument`, `Session`, `PlanDetailsPlan`.

## Data Model

### Collections & Relationships
```
credentials (UserCredentials)
├── _id: string (UUID)
├── email: string (unique, lowercase)
├── password_hash: string
├── createdAt: Date
└── updatedAt: Date

documents (UserDocument)
├── _id: string (same UUID as credentials)
├── first_name: string
├── last_name: string
├── email: string
├── plan: "single" | "startup" | "business"
├── projects: UserDocumentProjects[]  (embedded references)
│   ├── id: string
│   ├── title: string
│   ├── createdAt: Date
│   ├── updatedAt: Date
│   └── creator: { uid: string, email: string }
├── createdAt: Date
└── updatedAt: Date

projects (Project)
├── _id: string (UUID)
├── title: string
├── project_key: string (auto-generated "atom-<base64>")
├── creator_uid: string (references credentials._id)
├── posts: Post[]  (embedded subdocuments)
│   ├── id: string (UUID)
│   ├── title: string
│   ├── author: string
│   ├── body: string (Markdown)
│   ├── image: string | null
│   ├── teaser: string
│   ├── keywords: string[]
│   ├── creator_uid: string
│   ├── createdAt: Date
│   └── updatedAt: Date
├── createdAt: Date
└── updatedAt: Date

sessions (Session)
├── _id: ObjectId (auto)
├── user_id: string
└── expires_at: Date
```

### Plan Limits
| Plan     | Price      | Max Projects | Max Posts | Max Body Length |
|----------|------------|-------------|-----------|-----------------|
| single   | Free       | 2           | 100       | 10,000 chars    |
| startup  | $3.99/mo   | 3           | 1,000     | 100,000 chars   |
| business | $11.99/mo  | 5           | 2,500     | 500,000 chars   |

*Note: "startup" and "business" plans are currently disabled (coming soon).*

## API / Routes

### Auth Routes
| Method | Path | Purpose | Auth |
|--------|------|---------|------|
| `POST` | `/api/auth/signup` | Register user (email, password, first_name, last_name) | None |
| `POST` | `/api/auth/signin` | Login (email, password) | None |
| `POST` | `/api/auth/signout` | Logout (invalidate session) | Session cookie |
| `DELETE` | `/api/auth/delete` | Delete user account (requires password) | Session cookie |
| `GET` | `/api/auth/user/get` | Get current user document | Session cookie |
| `PATCH` | `/api/auth/user/update` | Update first_name/last_name | Session cookie |

### Project Routes
| Method | Path | Purpose | Auth |
|--------|------|---------|------|
| `POST` | `/api/projects/create` | Create project (title) | Session cookie |
| `DELETE` | `/api/projects/delete?project_id=<id>` | Delete project | Session cookie |
| `GET` | `/api/projects/get/single?project_id=<id>` | Get project (dashboard use) | Session cookie OR Bearer key |
| `GET` | `/api/projects/get/single/client` | Get project for SDK (sanitized) | Bearer key |

### Post Routes
| Method | Path | Purpose | Auth |
|--------|------|---------|------|
| `POST` | `/api/posts/create?project_id=<id>` | Create post in project | Session cookie |
| `DELETE` | `/api/posts/delete?project_id=<id>&post_id=<id>` | Delete post | Session cookie |
| `PATCH` | `/api/posts/update?project_id=<id>&post_id=<id>` | Update post fields | Session cookie |
| `GET` | `/api/posts/get/single?post_id=<id>` | Get single post (public) | Bearer key |

### Response Format
All API routes return:
```typescript
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```

### Page Routes
| Path | Description |
|------|-------------|
| `/` | Marketing landing page |
| `/signin` | Login page |
| `/signup` | Registration page |
| `/pricing` | Pricing plans display |
| `/blog` | Blog listing (dogfoods atom-nextjs SDK) |
| `/blog/[id]` | Blog post page (dogfoods atom-nextjs SDK) |
| `/app` | Dashboard – projects list (protected) |
| `/app/projects/[id]` | Project detail – post editor (protected) |
| `/app/settings` | User settings (protected) |
| `/app/settings/billing` | Billing page – coming soon (protected) |

## Dependencies

### Core
- **next 14.1.0** — React framework (App Router)
- **react 18** / **react-dom 18** — UI library
- **typescript 5** — Type safety

### Database & Auth
- **mongoose 8.1.2** — MongoDB ODM
- **lucia 3.1.1** — Session-based authentication library
- **@lucia-auth/adapter-mongodb 1.0.2** — Lucia MongoDB session adapter
- **argon2 0.40.1** — Password hashing (via `oslo` for crypto utilities)
- **oslo 1.1.1** — Cryptographic utilities used by Lucia

### Rate Limiting & Caching
- **@upstash/redis 1.28.4** — Serverless Redis client
- **@upstash/ratelimit 1.0.1** — Sliding window rate limiter

### UI
- **@radix-ui/** — Primitive UI components (dialog, dropdown-menu, alert-dialog, popover, label, slot)
- **class-variance-authority** / **clsx** / **tailwind-merge** — Utility-first styling helpers (shadcn/ui)
- **tailwindcss 3** + **tailwindcss-animate** + **@tailwindcss/typography** — CSS framework
- **lucide-react** / **react-icons** — Icon libraries
- **framer-motion** — Animations
- **@uiw/react-md-editor** — Markdown editor component
- **react-syntax-highlighter** — Code highlighting on landing page

### Forms & State
- **react-hook-form 7** + **@hookform/resolvers** — Form management
- **zod 3** — Schema validation
- **zustand 4** — State management (imported but minimal usage visible)
- **@tanstack/react-table 8** — Data table for projects listing
- **@tanstack/react-query 5** — Query management (imported but minimal usage visible)

### Content
- **next-mdx-remote 4** / **@mdx-js/loader** / **@next/mdx** — MDX rendering
- **react-markdown 9** / **remark-gfm** / **rehype-sanitize** / **rehype-katex** / **remark-math** — Markdown processing
- **sanitize-html** — HTML sanitization

### HTTP & Utilities
- **axios 1.6.7** — HTTP client for API calls
- **uuid 9** — UUID generation for IDs
- **react-hot-toast** — Toast notifications
- **react-tweet** — Twitter embed component

### SDK Package (`atom-nextjs`)
- **next-mdx-remote** — MDX compilation (RSC)
- **remark-gfm 3.0.0** / **rehype-sanitize** — Markdown plugins
- **react-loading-skeleton** — Loading skeletons
- Built with **tsdx** (TypeScript Development Extension)

## Build & Run

### Prerequisites
- Node.js ≥ 18
- MongoDB instance
- Upstash Redis instance

### Environment Variables
```bash
HASH_SALT="<secret>"              # Salt appended to passwords before Argon2 hashing
MONGO_DB_URI="<mongodb-uri>"      # MongoDB connection string
ATOM_PROJECT_KEY="<project-key>"  # Project key for the dogfood blog
UPSTASH_REDIS_REST_URL="<url>"    # Upstash Redis REST URL
UPSTASH_REDIS_REST_TOKEN="<tok>"  # Upstash Redis REST token
ENV="dev"                         # "dev" or "prod" (controls baseAPIRoute)
NEXT_PUBLIC_ENV="dev"             # Client-side env flag
```

### Scripts
```bash
npm run dev      # Start Next.js development server on localhost:3000
npm run build    # Production build
npm run start    # Start production server
npm run lint     # ESLint check
```

### SDK Local Development
```bash
cd packages/atom-nextjs
npm link              # Make package available locally
cd ../..
npm link atom-nextjs  # Link local version into main app
```

### SDK Build
```bash
cd packages/atom-nextjs
npm run build    # tsdx build → outputs to dist/
npm run start    # tsdx watch mode
npm run test     # tsdx test (passWithNoTests)
```

## Patterns & Conventions

### Code Organization
- **`@/*` path alias** — Maps to project root; used everywhere (e.g., `@/lib/types`, `@/components/ui/button`).
- **Server vs Client separation** — Server-only code lives in `lib/server/`; client API wrappers in `lib/client/`. Components are marked `"use client"` explicitly when needed.
- **Component colocation** — Components are organized by type: `containers/`, `forms/`, `modals/`, `nav/`, `sidebars/`, `pages/`, `cards/`, `tables/`, `ui/`.

### API Pattern
- All routes return `ApiResponse<T>` with `{ success, message, response }`.
- Error handling: try/catch blocks; errors thrown as `new Error(msg)`, caught and returned as `{ success: false, message: err.message }`.
- Auth check pattern: `await connectToDatabase()` → `await validateRequest()` → check `user` → proceed or throw.
- MongoDB duplicate key errors (code `11000`) are caught and mapped to user-friendly messages.

### Authentication Pattern
- Session-based auth via Lucia library.
- Session validation is cached per request using React `cache()`.
- Protected pages use the `ProtectedRoute` server component wrapper.
- API routes manually call `validateRequest()`.
- Public SDK endpoints use `Authorization: Bearer <project_key>` header.

### Form Validation
- All forms use **React Hook Form** + **Zod** schemas.
- Schemas define max lengths matching server-side constants (`maxInputLength = 30`, `projectTitleMaxLength = 50`).

### Error Handling
- Client-side: errors displayed via `toast.error()` from `react-hot-toast`.
- Server-side: errors logged to `console.log(err)` and returned as JSON.
- No global error boundary; per-route try/catch.

### Naming Conventions
- Files: camelCase for utilities (`fetchUser.ts`), PascalCase for components (`ProjectPage.tsx`).
- Types: PascalCase (`UserDocument`, `ApiResponse`).
- Constants file is misspelled as `contants.tsx` (not `constants`) — maintained throughout for import consistency.
- Route files use Next.js App Router convention: `route.ts` in directory-based paths.

### Database Operations
- Mongoose transactions used for multi-collection writes (`mongoose.startSession()` + `withTransaction()`).
- Posts are embedded arrays within project documents — updated via `$push`, `$pull`, and positional `$set` operators.
- All `_id` fields use string UUIDs (not ObjectId).

### Styling
- TailwindCSS with shadcn/ui design system (CSS variables in `globals.css`).
- `cn()` utility from `lib/utils.ts` for conditional class merging.
- Dark mode configured but not actively used.
- Montserrat font loaded via `next/font/google`.
