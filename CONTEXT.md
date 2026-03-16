Now I have all the information needed. Let me produce the CONTEXT.md:

# Atom CMS

## Overview
Atom is a content management system (CMS) built specifically for Next.js developers. It allows users to create, edit, and publish blog posts and articles through a dashboard web application, and then embed those blogs into their own Next.js sites using the companion `atom-nextjs` npm package. The tech stack is **Next.js 14** (App Router), **TypeScript**, **MongoDB** (via Mongoose), **Lucia** for session-based authentication, **Upstash Redis** for API rate limiting, **Tailwind CSS** + **shadcn/ui** for styling, **Zod** + **React Hook Form** for form validation, and **Argon2** for password hashing.

## File Tree
```
context/
├── randomcode.ts                          # Standalone hello-world script
├── test-code/
│   ├── .eslintrc.json
│   ├── .gitignore
│   ├── LICENSE                            # Apache 2.0
│   ├── README.md
│   ├── bun.lock
│   ├── components.json                    # shadcn/ui config
│   ├── middleware.ts                       # Next.js middleware (rate limiting)
│   ├── next.config.mjs
│   ├── package.json
│   ├── package-lock.json
│   ├── postcss.config.js
│   ├── tailwind.config.ts
│   ├── tsconfig.json
│   │
│   ├── app/
│   │   ├── favicon.ico
│   │   ├── globals.css                    # Tailwind + CSS variables
│   │   ├── layout.tsx                     # Root layout (Montserrat font, Toaster)
│   │   ├── page.tsx                       # Landing page / marketing home
│   │   ├── robots.ts                      # SEO robots.txt
│   │   ├── sitemap.ts                     # Dynamic sitemap generation
│   │   │
│   │   ├── signin/
│   │   │   └── page.tsx                   # Sign-in page
│   │   ├── signup/
│   │   │   └── page.tsx                   # Sign-up page
│   │   ├── pricing/
│   │   │   └── page.tsx                   # Pricing plans page
│   │   ├── blog/
│   │   │   ├── page.tsx                   # Blog listing (uses atom-nextjs)
│   │   │   └── [id]/
│   │   │       └── page.tsx               # Single blog post (uses atom-nextjs)
│   │   │
│   │   ├── app/                           # Authenticated dashboard area
│   │   │   ├── layout.tsx                 # ProtectedRoute wrapper
│   │   │   ├── page.tsx                   # Projects list (dashboard home)
│   │   │   ├── projects/
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx           # Single project editor
│   │   │   └── settings/
│   │   │       ├── page.tsx               # User settings
│   │   │       └── billing/
│   │   │           └── page.tsx           # Billing (coming soon)
│   │   │
│   │   └── api/
│   │       ├── auth/
│   │       │   ├── signup/route.ts        # POST - user registration
│   │       │   ├── signin/route.ts        # POST - user login
│   │       │   ├── signout/route.ts       # POST - user logout
│   │       │   ├── delete/route.ts        # DELETE - delete user account
│   │       │   └── user/
│   │       │       ├── get/route.ts       # GET - fetch current user
│   │       │       └── update/route.ts    # PATCH - update user profile
│   │       ├── posts/
│   │       │   ├── create/route.ts        # POST - create blog post
│   │       │   ├── delete/route.ts        # DELETE - delete blog post
│   │       │   ├── update/route.ts        # PATCH - update blog post
│   │       │   └── get/
│   │       │       └── single/route.ts    # GET - get single post (by project key)
│   │       └── projects/
│   │           ├── create/route.ts        # POST - create project
│   │           ├── delete/route.ts        # DELETE - delete project
│   │           └── get/
│   │               └── single/
│   │                   ├── route.ts       # GET - get project (auth or key)
│   │                   └── client/route.ts # GET - get project for public clients
│   │
│   ├── components/
│   │   ├── cards/
│   │   │   └── PricingPlanCard.tsx
│   │   ├── containers/
│   │   │   ├── AppContainer.tsx           # Dashboard layout with sidebar
│   │   │   ├── MainContainer.tsx          # Public page layout with navbar/footer
│   │   │   └── ProtectedRoute.tsx         # Server-side auth guard
│   │   ├── forms/
│   │   │   ├── LoginForm.tsx
│   │   │   └── SignupForm.tsx
│   │   ├── misc/
│   │   │   ├── NpmPackageComponent.tsx    # Copyable npm install command
│   │   │   └── tracing-beam.tsx           # Aceternity UI component
│   │   ├── modals/
│   │   │   ├── CreatePostModal.tsx        # Dialog for creating new posts
│   │   │   └── DeleteUserModal.tsx        # Dialog for account deletion
│   │   ├── nav/
│   │   │   └── Navbar.tsx                 # Public site navbar (async server)
│   │   ├── pages/
│   │   │   ├── projects/
│   │   │   │   ├── ProjectComponent.tsx   # Project editor shell
│   │   │   │   ├── ProjectFormComponent.tsx # Post edit form (markdown editor)
│   │   │   │   └── ProjectPage.tsx        # Projects list with create dialog
│   │   │   └── settings/
│   │   │       └── SettingsForm.tsx        # User profile edit form
│   │   ├── sidebars/
│   │   │   ├── AppSidebarNav.tsx          # Dashboard sidebar navigation
│   │   │   └── ProjectComponentSidebar.tsx # Post list sidebar in project editor
│   │   ├── tables/
│   │   │   └── UserDocumentProjects/
│   │   │       ├── columns.tsx            # TanStack Table column definitions
│   │   │       └── table.tsx              # TanStack Table component
│   │   └── ui/                            # shadcn/ui primitives
│   │       ├── alert-dialog.tsx
│   │       ├── button.tsx
│   │       ├── carousel.tsx
│   │       ├── dialog.tsx
│   │       ├── dropdown-menu.tsx
│   │       ├── form.tsx
│   │       ├── input.tsx
│   │       ├── label.tsx
│   │       ├── markdown-editor.tsx         # @uiw/react-md-editor wrapper
│   │       ├── popover.tsx
│   │       ├── sticky-scroll-reveal.tsx
│   │       ├── table.tsx
│   │       └── textarea.tsx
│   │
│   ├── lib/
│   │   ├── contants.tsx                   # App constants (plans, nav, routes)
│   │   ├── types.ts                       # Core TypeScript types
│   │   ├── utils.ts                       # cn() Tailwind utility
│   │   ├── utils/
│   │   │   └── validateEmail.ts           # Email regex validator
│   │   ├── client/                        # Client-side API wrappers (axios)
│   │   │   ├── auth/
│   │   │   │   ├── deleteUser.ts
│   │   │   │   ├── loginUser.ts
│   │   │   │   ├── signoutUser.ts
│   │   │   │   ├── signupUser.ts
│   │   │   │   └── updateUser.ts
│   │   │   ├── posts/
│   │   │   │   ├── createPost.ts
│   │   │   │   ├── deletePost.ts
│   │   │   │   └── updatePost.ts
│   │   │   └── projects/
│   │   │       ├── createProject.ts
│   │   │       └── deleteProject.ts
│   │   └── server/                        # Server-side utilities
│   │       ├── encoding/
│   │       │   ├── encodePassword.ts      # Argon2 hash with salt
│   │       │   └── isPasswordValid.ts     # Argon2 verify
│   │       ├── functions/
│   │       │   ├── projects/
│   │       │   │   └── getProject.ts      # SSR project fetch via cookies
│   │       │   └── user/
│   │       │       └── fetchUser.ts       # SSR user fetch via cookies
│   │       ├── lucia/
│   │       │   ├── init.ts                # Lucia auth setup with MongoDB adapter
│   │       │   └── functions/
│   │       │       └── validate-request.ts # Cached session validation
│   │       ├── mongo/
│   │       │   ├── init.ts                # Mongoose connection + model refs
│   │       │   └── types/
│   │       │       ├── userCredentials.ts # Mongoose schema for credentials
│   │       │       ├── userDocuments.ts   # Mongoose schema for user docs
│   │       │       ├── userProjects.ts    # Mongoose schema for projects/posts
│   │       │       └── userSessions.ts    # Mongoose schema for sessions
│   │       ├── redis/
│   │       │   └── init.ts               # Upstash Redis + rate limiter init
│   │       └── utils/
│   │           ├── generateProjectKey.ts  # Random base64 key generator
│   │           ├── validateProjectKey.ts  # Project key lookup
│   │           └── validateRequestFetchUser.ts # Combined auth + user fetch
│   │
│   ├── bruno/                             # Bruno API client collection
│   │   ├── bruno.json
│   │   ├── environments/
│   │   │   └── Development.bru
│   │   └── Routes/
│   │       ├── Create Post.bru
│   │       ├── Create Project.bru
│   │       ├── Delete Post.bru
│   │       ├── Delete Project.bru
│   │       ├── Get Post.bru
│   │       ├── Get Project.bru
│   │       └── Get User.bru
│   │
│   ├── packages/
│   │   └── atom-nextjs/                   # Published npm package (SDK)
│   │       ├── package.json
│   │       ├── tsconfig.json
│   │       ├── yarn.lock
│   │       ├── package-lock.json
│   │       ├── README.md
│   │       ├── LICENSE
│   │       ├── .gitignore
│   │       ├── .github/workflows/
│   │       │   ├── main.yml
│   │       │   └── size.yml
│   │       └── src/
│   │           ├── index.tsx              # Package entry - exports all components
│   │           ├── components/
│   │           │   ├── Atom.tsx           # Full blog post renderer
│   │           │   ├── AtomArticleSkeleton.tsx
│   │           │   ├── AtomBody.tsx       # MDX body renderer
│   │           │   ├── AtomLoadingSkeleton.tsx
│   │           │   ├── AtomPage.tsx       # Blog listing page component
│   │           │   └── AtomPostCard.tsx   # Individual post card
│   │           └── lib/
│   │               ├── constants.ts       # Base API URL
│   │               ├── types.ts           # SDK-specific types
│   │               └── client/
│   │                   ├── generatePostMetadata.ts
│   │                   ├── generateSitemap.ts
│   │                   ├── getPost.ts
│   │                   └── getProject.ts
│   │
│   └── public/
│       ├── atom-black.svg
│       ├── next.svg
│       └── vercel.svg
```

## Architecture

### High-Level Flow
The codebase consists of two parts:

1. **Main Application (`context/test-code/`)** — A Next.js 14 App Router application that serves as both:
   - A **marketing site** (landing, pricing, blog pages) for public visitors
   - An **authenticated dashboard** (under `/app/`) where users manage projects and blog posts

2. **SDK Package (`packages/atom-nextjs/`)** — An npm package that consumers install to render Atom-managed blog posts in their own Next.js applications using server-side components.

### Data Flow
```
User Dashboard ──► Client API wrappers (axios) ──► Next.js API Routes ──► MongoDB
                                                                           ▲
Public Blog (atom-nextjs SDK) ──► fetch() with Bearer token ──────────────┘
```

- **Authentication**: Lucia v3 with MongoDB adapter. Session cookies are set on sign-in/sign-up and validated via `validateRequest()` (cached per request using React `cache()`).
- **Rate Limiting**: All `/api/*` routes pass through Next.js middleware that applies Upstash Redis sliding-window rate limiting (30 requests/minute per IP).
- **Authorization**: API routes verify the authenticated user owns the project/post via `creator_uid` checks. Public read endpoints use Bearer token (project key) authentication.

### Directory Responsibilities

| Directory | Purpose |
|-----------|---------|
| `app/` | Next.js page routes and API route handlers |
| `app/api/` | REST API endpoints (auth, posts, projects) |
| `app/app/` | Protected dashboard routes (wrapped in ProtectedRoute) |
| `components/` | React components (forms, modals, layouts, tables, UI primitives) |
| `components/ui/` | shadcn/ui components (auto-generated via CLI) |
| `lib/client/` | Client-side API wrapper functions using axios |
| `lib/server/` | Server-side logic (auth, DB, encoding, utilities) |
| `lib/server/mongo/` | Mongoose connection, model refs, and schemas |
| `lib/server/lucia/` | Lucia auth initialization and session validation |
| `lib/server/redis/` | Upstash Redis + rate limiter setup |
| `packages/atom-nextjs/` | Published npm SDK for rendering Atom blogs |
| `bruno/` | Bruno API client collection for manual API testing |

## Key Files

### Configuration
- **`package.json`** — Project metadata, scripts (`dev`, `build`, `start`, `lint`), and all dependencies.
- **`next.config.mjs`** — Disables caching (no-store headers) for `/`, `/app/*`, and `/api/*` routes.
- **`tsconfig.json`** — TypeScript config with `@/*` path alias mapping to project root.
- **`tailwind.config.ts`** — Extended Tailwind config with shadcn/ui theme, CSS variable colors, typography plugin, and custom color variable injection.
- **`components.json`** — shadcn/ui configuration (default style, RSC enabled, `@/` aliases).
- **`middleware.ts`** — Next.js middleware: rate-limits all `/api/*` requests using Upstash Redis (30 req/min per IP).

### Core Types & Constants
- **`lib/types.ts`** — All core TypeScript types: `UserCredentials`, `Post`, `Project`, `UserDocument`, `Session`, `Plan`, `PlanDetailsPlan`, etc.
- **`lib/contants.tsx`** — App constants: plan tiers (`single`/`startup`/`business`), nav options, `baseAPIRoute`, `planDetails` array with feature limits, `maxInputLength`, `projectTitleMaxLength`.

### Server Infrastructure
- **`lib/server/mongo/init.ts`** — Mongoose connection function and model references (`UserCredentialsRef`, `UserDocumentsRef`, `ProjectsRef`, `SessionRef`).
- **`lib/server/mongo/types/`** — Mongoose schemas for credentials, user documents, projects (with embedded posts), and sessions.
- **`lib/server/lucia/init.ts`** — Lucia auth instance with MongoDB adapter, session cookie config, and `getUserAttributes` extracting email.
- **`lib/server/lucia/functions/validate-request.ts`** — Core auth function: reads session cookie, validates via Lucia, refreshes session if needed. Cached with React `cache()`.
- **`lib/server/redis/init.ts`** — Upstash Redis client and `Ratelimit` instance (sliding window, 30/min).
- **`lib/server/encoding/encodePassword.ts`** — Hashes password with Argon2 + environment salt.
- **`lib/server/encoding/isPasswordValid.ts`** — Verifies password against Argon2 hash.
- **`lib/server/utils/generateProjectKey.ts`** — Generates `atom-` prefixed base64 random project API keys.

### API Routes
- **`app/api/auth/signup/route.ts`** — User registration with MongoDB transaction (creates credentials + user document), creates Lucia session. Defines `ApiResponse<T>` type used across all routes.
- **`app/api/auth/signin/route.ts`** — Login: verifies credentials, creates session.
- **`app/api/auth/signout/route.ts`** — Logout: invalidates session, clears cookie.
- **`app/api/auth/delete/route.ts`** — Account deletion: password verification, transactional deletion of credentials, projects, documents, sessions.
- **`app/api/auth/user/get/route.ts`** — Fetch current user document.
- **`app/api/auth/user/update/route.ts`** — Update first/last name.
- **`app/api/posts/create/route.ts`** — Create post within a project (checks plan limits on body length).
- **`app/api/posts/delete/route.ts`** — Delete post from project.
- **`app/api/posts/update/route.ts`** — Partial update post fields.
- **`app/api/posts/get/single/route.ts`** — Public endpoint: fetch post by project key (Bearer auth). Adds watermark for free plan.
- **`app/api/projects/create/route.ts`** — Create project with transaction (creates project + updates user document). Checks plan project limits.
- **`app/api/projects/delete/route.ts`** — Delete project with transaction.
- **`app/api/projects/get/single/route.ts`** — Get project: supports both Bearer key auth (public) and session auth (dashboard).
- **`app/api/projects/get/single/client/route.ts`** — Public client endpoint: returns sanitized project data (no body, no project key) via Bearer auth.

### Pages
- **`app/page.tsx`** — Marketing landing page with code examples, demo iframe, and CTA.
- **`app/signin/page.tsx`** / **`app/signup/page.tsx`** — Auth pages with form components.
- **`app/blog/page.tsx`** — Public blog listing using `atom-nextjs` SDK's `AtomPage` component.
- **`app/blog/[id]/page.tsx`** — Single post view using `atom-nextjs` SDK's `Atom` component + `generatePostMetadata`.
- **`app/app/page.tsx`** — Dashboard home: shows user's projects in a data table.
- **`app/app/projects/[id]/page.tsx`** — Project editor: sidebar with post list + post editing form.

### Key Components
- **`components/containers/ProtectedRoute.tsx`** — Server component that validates session and redirects to `/signin` if unauthenticated.
- **`components/containers/AppContainer.tsx`** — Dashboard layout with `AppSidebarNav` + main content area.
- **`components/containers/MainContainer.tsx`** — Public page layout with `Navbar` + footer.
- **`components/pages/projects/ProjectComponent.tsx`** — Client component managing post selection state within a project.
- **`components/pages/projects/ProjectFormComponent.tsx`** — Rich post editor with markdown editor, Zod validation, update/delete functionality.
- **`components/sidebars/ProjectComponentSidebar.tsx`** — Post list sidebar with "Copy project key" and "Create new post" buttons.
- **`components/forms/LoginForm.tsx`** / **`SignupForm.tsx`** — Zod-validated forms using React Hook Form.
- **`components/modals/CreatePostModal.tsx`** — Dialog for creating new blog posts with all fields.
- **`components/modals/DeleteUserModal.tsx`** — Confirmation dialog requiring password to delete account.

### SDK Package (`packages/atom-nextjs/`)
- **`src/index.tsx`** — Exports: `Atom`, `AtomBody`, `AtomPage`, `AtomPostCard`, `AtomLoadingSkeleton`, `AtomArticleSkeleton`, `generatePostMetadata`, `getPost`, `getProject`, `generateSitemap`.
- **`src/components/Atom.tsx`** — Async server component that fetches a post and renders it with MDX (title, image, author, date, body).
- **`src/components/AtomBody.tsx`** — Compiles MDX with `next-mdx-remote/rsc`, `remark-gfm`, `rehype-sanitize`.
- **`src/components/AtomPage.tsx`** — Fetches all posts for a project and renders `AtomPostCard` grid.
- **`src/lib/client/getPost.ts`** / **`getProject.ts`** — Fetch functions using Bearer token auth against the Atom API.
- **`src/lib/client/generatePostMetadata.ts`** — Generates Next.js `Metadata` from a post for SEO.
- **`src/lib/client/generateSitemap.ts`** — Generates sitemap entries from all posts in a project.

## Data Model

### MongoDB Collections & Schemas

**`credentials`** (`UserCredentials`)
| Field | Type | Notes |
|-------|------|-------|
| `_id` | String (UUID) | User ID, set manually |
| `email` | String | Unique, lowercase, trimmed |
| `password_hash` | String | Argon2 hash |
| `createdAt` | Date | Auto (timestamps) |
| `updatedAt` | Date | Auto (timestamps) |

**`documents`** (`UserDocument`)
| Field | Type | Notes |
|-------|------|-------|
| `_id` | String (UUID) | Matches credentials `_id` |
| `first_name` | String | |
| `last_name` | String | |
| `email` | String | |
| `plan` | String | Enum: `single`, `startup`, `business` |
| `projects` | Array\<UserDocumentProjects\> | Embedded project references |
| `createdAt` | Date | Auto |
| `updatedAt` | Date | Auto |

**`projects`** (`Project`)
| Field | Type | Notes |
|-------|------|-------|
| `_id` | String (UUID) | Project ID |
| `title` | String | |
| `project_key` | String | `atom-` + random base64 |
| `creator_uid` | String | References user `_id` |
| `posts` | Array\<Post\> | Embedded posts |
| `createdAt` | Date | Auto |
| `updatedAt` | Date | Auto |

**Embedded `Post`** (within `projects.posts`)
| Field | Type | Notes |
|-------|------|-------|
| `id` | String (UUID) | Post ID |
| `title` | String | |
| `author` | String | |
| `body` | String | Markdown/MDX content |
| `image` | String \| null | Cover image URL |
| `teaser` | String | Summary text |
| `keywords` | String[] | Optional |
| `creator_uid` | String | |
| `createdAt` | Date | Auto |
| `updatedAt` | Date | Auto |

**`sessions`** (`Session`) — Managed by Lucia
| Field | Type |
|-------|------|
| `user_id` | String |
| `expires_at` | Date |

### Relationships
- A **User** has one `credentials` doc and one `documents` doc sharing the same `_id`.
- A **UserDocument** embeds an array of `UserDocumentProjects` (project references with `id`, `title`, `creator`).
- A **Project** embeds an array of `Post` objects (posts are not a separate collection).
- When a project is created/deleted, both `projects` and `documents` collections are updated in a MongoDB transaction.

### Plan Limits
| Plan | Price | Max Projects | Max Posts | Max Body Length |
|------|-------|-------------|-----------|----------------|
| single | Free | 2 | 100 | 10,000 chars |
| startup | $3.99/mo | 3 | 1,000 | 100,000 chars |
| business | $11.99/mo | 5 | 2,500 | 500,000 chars |

(Startup and Business plans are currently `disabled: true`.)

## API / Routes

All API routes return `ApiResponse<T>`: `{ success: boolean; message: string | null; response: T }`.

### Authentication
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/signup` | None | Register new user. Body: `{ email, password, first_name, last_name }` |
| POST | `/api/auth/signin` | None | Login. Body: `{ email, password }` |
| POST | `/api/auth/signout` | Session | Logout current user |
| DELETE | `/api/auth/delete` | Session | Delete account. Body: `{ password }` |
| GET | `/api/auth/user/get` | Session | Get current user document |
| PATCH | `/api/auth/user/update` | Session | Update profile. Body: `{ first_name?, last_name? }` |

### Posts
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/posts/create?project_id=` | Session | Create post. Body: `{ title, author, body, image?, keywords?, teaser }` |
| PATCH | `/api/posts/update?project_id=&post_id=` | Session | Update post. Body: `{ title?, author?, body?, teaser?, keywords?, image? }` |
| DELETE | `/api/posts/delete?project_id=&post_id=` | Session | Delete post |
| GET | `/api/posts/get/single?post_id=` | Bearer (project key) | Public: Get single post (adds watermark on free plan) |

### Projects
| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/projects/create` | Session | Create project. Body: `{ title }` |
| DELETE | `/api/projects/delete?project_id=` | Session | Delete project |
| GET | `/api/projects/get/single?project_id=` | Session or Bearer | Get project (full data) |
| GET | `/api/projects/get/single/client` | Bearer (project key) | Public: Get project with sanitized posts (no body) |

### Page Routes
| Path | Description |
|------|-------------|
| `/` | Marketing landing page |
| `/signin` | Login page |
| `/signup` | Registration page |
| `/pricing` | Pricing plans display |
| `/blog` | Public blog listing (uses atom-nextjs SDK) |
| `/blog/[id]` | Public blog post (uses atom-nextjs SDK) |
| `/app` | Dashboard: projects list (protected) |
| `/app/projects/[id]` | Project editor with post sidebar (protected) |
| `/app/settings` | User profile settings (protected) |
| `/app/settings/billing` | Billing page — placeholder (protected) |

## Dependencies

### Core Framework
- **next** `14.1.0` — React framework (App Router)
- **react** / **react-dom** `^18` — UI library
- **typescript** `^5` — Type system

### Database & Auth
- **mongoose** `^8.1.2` — MongoDB ODM
- **lucia** `^3.1.1` — Session-based authentication library
- **@lucia-auth/adapter-mongodb** `^1.0.2` — Lucia MongoDB adapter
- **argon2** `^0.40.1` — Password hashing
- **oslo** `^1.1.1` — Utility library for auth (Lucia dependency)

### Rate Limiting / Caching
- **@upstash/redis** `^1.28.4` — Serverless Redis client
- **@upstash/ratelimit** `^1.0.1` — Rate limiting middleware

### UI & Styling
- **tailwindcss** `^3.3.0` + **tailwindcss-animate** — CSS framework
- **@radix-ui/react-*` (dialog, dropdown-menu, label, popover, slot, alert-dialog)** — Headless UI primitives (via shadcn/ui)
- **class-variance-authority** + **clsx** + **tailwind-merge** — Style utilities
- **lucide-react** — Icons
- **react-icons** — Additional icons
- **framer-motion** — Animations

### Forms & Validation
- **react-hook-form** `^7.50.1` + **@hookform/resolvers** — Form state management
- **zod** `^3.22.4` — Schema validation

### Content & Markdown
- **@uiw/react-md-editor** `^4.0.4` — Markdown editor component
- **next-mdx-remote** `^4.4.1` — MDX rendering for server components
- **react-markdown** `^9.0.1` — Markdown rendering
- **react-syntax-highlighter** `^15.5.0` — Code block highlighting
- **remark-gfm**, **remark-math**, **rehype-katex**, **rehype-sanitize** — MDX plugins

### State & Data
- **zustand** `^4.5.0` — Client state management
- **@tanstack/react-query** `^5.27.5` — Server state management
- **@tanstack/react-table** `^8.13.2` — Table UI library
- **axios** `^1.6.7` — HTTP client

### Other
- **react-hot-toast** `^2.4.1` — Toast notifications
- **uuid** `^9.0.1` — UUID generation
- **atom-nextjs** `^0.3.1` — The project's own published SDK (dogfooded on the blog pages)
- **react-tweet** `^3.2.0` — Embedded tweets
- **sanitize-html** `^2.12.1` — HTML sanitization

## Build & Run

### Prerequisites
- Node.js with npm/bun
- MongoDB instance
- Upstash Redis instance

### Environment Variables
```bash
HASH_SALT="..."              # Salt appended to passwords before Argon2 hashing
MONGO_DB_URI="..."           # MongoDB connection string
ATOM_PROJECT_KEY="..."       # Project key for the self-hosted blog
UPSTASH_REDIS_REST_URL="..." # Upstash Redis URL
UPSTASH_REDIS_REST_TOKEN="..." # Upstash Redis auth token
NEXT_PUBLIC_ENV="dev"|"prod" # Controls baseAPIRoute (localhost vs production)
ENV="dev"|"prod"             # Environment flag
```

### Scripts
```bash
npm run dev     # Start Next.js dev server
npm run build   # Production build
npm run start   # Start production server
npm run lint    # ESLint
```

### Running the atom-nextjs SDK locally
```bash
cd packages/atom-nextjs
npm link
cd ../..
npm link atom-nextjs
```

### SDK Package Scripts
```bash
cd packages/atom-nextjs
npm run start    # tsdx watch mode
npm run build    # tsdx build
npm run test     # tsdx test
npm run lint     # tsdx lint
```

## Patterns & Conventions

### API Response Pattern
All API routes return a typed `ApiResponse<T>`:
```typescript
type ApiResponse<T = null> = {
  success: boolean;
  message: string | null;
  response: T;
};
```
Client wrappers throw on `!data.success` and extract `data.response`.

### Error Handling
- **API Routes**: Try/catch blocks; errors returned as `{ success: false, message: err.message }` — never thrown to client as 500s.
- **Client functions**: Throw errors on failure; components catch and display via `toast.error()`.
- **MongoDB duplicate key** (`err.code === 11000`): Special handling in signup route.

### Authentication Pattern
- Session-based using Lucia v3 with cookies.
- `validateRequest()` is cached per request via React's `cache()`.
- Protected pages use `ProtectedRoute` server component (checks session, redirects to `/signin`).
- API routes call `validateRequest()` directly and throw if no user.
- Public read APIs use `Authorization: Bearer <project_key>` header instead of sessions.

### Database Transactions
MongoDB transactions (`mongoose.startSession()` + `withTransaction()`) are used for multi-document operations:
- User signup (create credentials + user document)
- Project creation (create project + update user document)
- Project deletion (delete project + update user document)
- Account deletion (delete credentials, projects, documents, sessions)

### Component Patterns
- **Server Components** (default): Pages, layouts, `Navbar`, `MainContainer`, `ProtectedRoute`, all SDK components.
- **Client Components** (`"use client"`): Forms, interactive editors, sidebars, modals, tables — anything with state, event handlers, or `useRouter`.
- **Form Validation**: Zod schemas + `zodResolver` with React Hook Form.
- **Toast Notifications**: `react-hot-toast` for success/error feedback. Success toasts often in client API wrappers; error toasts in component catch blocks.

### File Naming
- Components use PascalCase filenames: `ProjectComponent.tsx`, `LoginForm.tsx`.
- Utility/library files use camelCase: `createPost.ts`, `validateEmail.ts`.
- API routes follow Next.js App Router convention: `route.ts` in directory-based paths.
- Note: constants file is misspelled as `contants.tsx` (not `constants`).

### Path Aliases
- `@/*` maps to the project root, used extensively: `@/lib/types`, `@/components/ui/button`, etc.

### Styling
- Tailwind CSS with CSS custom properties (HSL color system via shadcn/ui).
- `cn()` utility (clsx + tailwind-merge) for conditional class merging.
- shadcn/ui components in `components/ui/` — default style, RSC-compatible.
- Montserrat font loaded via `next/font/google`.

### State Management
- Server state handled via SSR (async server components fetching data directly).
- Client state via React `useState` in most components.
- `zustand` is listed as a dependency but not visibly used in the current codebase.
- `@tanstack/react-query` is listed but not visibly used (data fetching is done via direct axios calls + `router.refresh()`).
