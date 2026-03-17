# Context Index

## Overview
Atom is a Content Management System (CMS) built for Next.js that enables users to create, edit, and publish blog posts via a web dashboard, then render them in external Next.js applications through an npm SDK package (`atom-nextjs`).

The core stack is **Next.js 14 (App Router)**, **TypeScript**, **MongoDB (Mongoose)**, **Lucia Auth** for session-based authentication, **Upstash Redis** for API rate limiting, **TailwindCSS** with **shadcn/ui** components, and **Zod** + **React Hook Form** for validation.

The project ships as two parts: a full-stack dashboard application where users manage their blog content, and a published npm package (`atom-nextjs`) that consumer Next.js apps install to fetch and render that content. The SDK uses `tsdx` for building and `next-mdx-remote` for MDX rendering. The project is licensed under Apache 2.0 and authored by incogiscool (Adam El Taha).

## Architecture
The codebase lives in `context/test-code/` and has two main parts:

1. **Main App** — A Next.js 14 App Router application serving both the marketing site, auth flows, a protected dashboard for managing projects/posts, and a public API for the SDK.
2. **SDK Package** (`packages/atom-nextjs/`) — A published npm package providing React server components and helper functions for rendering Atom blog content in consumer Next.js apps.

### Directory Structure
- `app/` — Next.js App Router pages and API routes
  - `app/api/` — REST API endpoints (auth, posts, projects)
  - `app/app/` — Protected dashboard pages (projects, settings, billing)
  - `app/blog/` — Public blog pages (using the SDK itself)
  - `app/signin/`, `app/signup/`, `app/pricing/` — Public pages
- `components/` — React components (containers, forms, modals, nav, pages, sidebars, tables, ui)
- `lib/` — Shared logic
  - `lib/types.ts` — Core TypeScript type definitions
  - `lib/contants.tsx` — Plans, nav options, API base URL, limits (note: typo in filename)
  - `lib/client/` — Client-side API wrapper functions (axios-based)
  - `lib/server/` — Server-side utilities (auth, DB, encoding, Redis)
- `packages/atom-nextjs/` — The SDK package
- `bruno/` — Bruno API collection for manual testing

### Data Flow
1. Users sign up/in via the dashboard (Lucia auth sessions stored in MongoDB)
2. Users create projects, each with a unique `project_key`
3. Users create/edit/delete posts within projects
4. External Next.js apps use the `atom-nextjs` SDK with the `project_key` as a Bearer token to fetch and render blog content via the public API

## Key Concepts

- **Project** — A container for blog posts, identified by a `project_key` (Bearer token for API access)
  - Defined in: `lib/types.ts:L39-L47`
  - Schema: `lib/server/mongo/types/userProjects.ts:L44-L68`
  - API CRUD: `app/api/projects/`

- **Post** — A blog post with title, author, MDX body, image, keywords, teaser
  - Defined in: `lib/types.ts:L11-L22`
  - Schema: `lib/server/mongo/types/userProjects.ts:L6-L42` (embedded in Project)
  - API CRUD: `app/api/posts/`

- **UserDocument** — User profile with name, email, plan, and project references
  - Defined in: `lib/types.ts:L49-L58`
  - Schema: `lib/server/mongo/types/userDocuments.ts:L53-L82`

- **UserCredentials** — Login credentials (email + argon2 password hash)
  - Defined in: `lib/types.ts:L3-L9`
  - Schema: `lib/server/mongo/types/userCredentials.ts:L6-L25`

- **Plan** — Subscription tier (`"single"` | `"startup"` | `"business"`) with resource limits
  - Defined in: `lib/types.ts:L24`
  - Plan details: `lib/contants.tsx:L38-L87`
  - Only `"single"` (free) is active; startup/business are `disabled: true`

- **ApiResponse<T>** — Standard API response shape used across all endpoints
  - Defined in: `app/api/auth/signup/route.ts:L17-L21`

- **SDK (atom-nextjs)** — Published npm package for consumer Next.js apps
  - Entry: `packages/atom-nextjs/src/index.tsx:L1-L23`
  - Key exports: `Atom`, `AtomPage`, `AtomBody`, `AtomPostCard`, `getPost`, `getProject`, `generatePostMetadata`, `generateSitemap`, `AtomLoadingSkeleton`, `AtomArticleSkeleton`

- **Rate Limiting** — Upstash Redis sliding window (30 requests/minute per IP) on `/api` routes
  - Defined in: `lib/server/redis/init.ts:L15-L18`
  - Applied in: `middleware.ts:L8-L37`

## API Surface

### Auth Routes
| Method | Path | Auth | Handler File | Line |
|--------|------|------|-------------|------|
| `POST` | `/api/auth/signup` | None | `app/api/auth/signup/route.ts` | L46 |
| `POST` | `/api/auth/signin` | None | `app/api/auth/signin/route.ts` | L19 |
| `POST` | `/api/auth/signout` | Session cookie | `app/api/auth/signout/route.ts` | L8 |
| `DELETE` | `/api/auth/delete` | Session cookie | `app/api/auth/delete/route.ts` | L18 |
| `GET` | `/api/auth/user/get` | Session cookie | `app/api/auth/user/get/route.ts` | L8 |
| `PATCH` | `/api/auth/user/update` | Session cookie | `app/api/auth/user/update/route.ts` | L12 |

### Project Routes
| Method | Path | Auth | Handler File | Line |
|--------|------|------|-------------|------|
| `POST` | `/api/projects/create` | Session cookie | `app/api/projects/create/route.ts` | L19 |
| `DELETE` | `/api/projects/delete?project_id=X` | Session cookie | `app/api/projects/delete/route.ts` | L15 |
| `GET` | `/api/projects/get/single?project_id=X` | Session cookie OR Bearer key | `app/api/projects/get/single/route.ts` | L8 |
| `GET` | `/api/projects/get/single/client` | Bearer key | `app/api/projects/get/single/client/route.ts` | L23 |

### Post Routes
| Method | Path | Auth | Handler File | Line |
|--------|------|------|-------------|------|
| `POST` | `/api/posts/create?project_id=X` | Session cookie | `app/api/posts/create/route.ts` | L22 |
| `DELETE` | `/api/posts/delete?project_id=X&post_id=Y` | Session cookie | `app/api/posts/delete/route.ts` | L6 |
| `PATCH` | `/api/posts/update?project_id=X&post_id=Y` | Session cookie | `app/api/posts/update/route.ts` | L35 |
| `GET` | `/api/posts/get/single?post_id=X` | Bearer key | `app/api/posts/get/single/route.ts` | L10 |

### Page Routes
| Path | Type | File |
|------|------|------|
| `/` | Public (marketing) | `app/page.tsx` |
| `/signin` | Public | `app/signin/page.tsx` |
| `/signup` | Public | `app/signup/page.tsx` |
| `/pricing` | Public | `app/pricing/page.tsx` |
| `/blog` | Public (SDK-rendered) | `app/blog/page.tsx` |
| `/blog/[id]` | Public (SDK-rendered) | `app/blog/[id]/page.tsx` |
| `/app` | Protected (dashboard) | `app/app/page.tsx` |
| `/app/projects/[id]` | Protected | `app/app/projects/[id]/page.tsx` |
| `/app/settings` | Protected | `app/app/settings/page.tsx` |
| `/app/settings/billing` | Protected | `app/app/settings/billing/page.tsx` |

### SDK Public API (atom-nextjs package)
| Export | Type | File |
|--------|------|------|
| `Atom` | Async server component (single post) | `packages/atom-nextjs/src/components/Atom.tsx:L11` |
| `AtomPage` | Async server component (post grid) | `packages/atom-nextjs/src/components/AtomPage.tsx:L5` |
| `AtomBody` | Async server component (MDX renderer) | `packages/atom-nextjs/src/components/AtomBody.tsx:L6` |
| `AtomPostCard` | Client component (post card) | `packages/atom-nextjs/src/components/AtomPostCard.tsx:L5` |
| `AtomLoadingSkeleton` | Loading skeleton (post grid) | `packages/atom-nextjs/src/components/AtomLoadingSkeleton.tsx:L5` |
| `AtomArticleSkeleton` | Loading skeleton (article) | `packages/atom-nextjs/src/components/AtomArticleSkeleton.tsx:L4` |
| `getPost` | Fetch function | `packages/atom-nextjs/src/lib/client/getPost.ts:L4` |
| `getProject` | Fetch function | `packages/atom-nextjs/src/lib/client/getProject.ts:L4` |
| `generatePostMetadata` | Next.js Metadata helper | `packages/atom-nextjs/src/lib/client/generatePostMetadata.ts:L4` |
| `generateSitemap` | Sitemap helper | `packages/atom-nextjs/src/lib/client/generateSitemap.ts:L3` |

## Data Model

### Core Types (all in `lib/types.ts`)
| Type | Lines | Description |
|------|-------|-------------|
| `UserCredentials` | L3-L9 | Login credentials (email, password_hash, timestamps, _id) |
| `Post` | L11-L22 | Blog post (title, author, body, image, keywords, teaser, creator_uid) |
| `Plan` | L24 | Union type: `"single"` \| `"startup"` \| `"business"` |
| `UserDocumentProjects` | L26-L32 | Denormalized project summary embedded in user doc |
| `UserDocumnetProjectsCreator` | L34-L37 | Creator reference (uid, email) — note typo in name |
| `Project` | L39-L47 | Full project with embedded posts array, project_key |
| `UserDocument` | L49-L58 | User profile with projects array, plan |
| `Session` | L60-L63 | Auth session (user_id, expires_at) |
| `NavOptionIds` | L65 | Sidebar nav option identifiers |
| `PlanDetailsPlan` | L67-L78 | Plan configuration with limits and features |

### SDK Types (`packages/atom-nextjs/src/lib/types.ts`)
| Type | Lines | Description |
|------|-------|-------------|
| `Post` | L1-L12 | Same as main Post type |
| `ClientPost` | L14-L22 | Sanitized post (no body, no creator_uid, no keywords) |
| `ClientProject` | L24-L30 | Sanitized project (no project_key, no creator_uid) |
| `ApiResponse<T>` | L31-L35 | Standard API response wrapper |

### API-specific Types
| Type | File | Lines |
|------|------|-------|
| `ApiResponse<T>` | `app/api/auth/signup/route.ts` | L17-L21 |
| `SignupRequestParams` | `app/api/auth/signup/route.ts` | L23-L28 |
| `SigninRequestParams` | `app/api/auth/signin/route.ts` | L14-L17 |
| `DeleteUserRequestBody` | `app/api/auth/delete/route.ts` | L14-L16 |
| `UpdateUserRequestParams` | `app/api/auth/user/update/route.ts` | L7-L10 |
| `CreatePostRequest` | `app/api/posts/create/route.ts` | L13-L20 |
| `UpdatePostRequestParams` | `app/api/posts/update/route.ts` | L15-L22 |
| `CreateProjectRequest` | `app/api/projects/create/route.ts` | L15-L17 |
| `DeleteProjectRequestParams` | `app/api/projects/delete/route.ts` | L11-L13 |
| `ClientPost` / `ClientProject` | `app/api/projects/get/single/client/route.ts` | L6-L21 |

### MongoDB Collections
| Collection | Model | Schema File |
|------------|-------|-------------|
| `credentials` | `UserCredentialsRef` | `lib/server/mongo/types/userCredentials.ts:L6-L25` |
| `documents` | `UserDocumentsRef` | `lib/server/mongo/types/userDocuments.ts:L53-L82` |
| `projects` | `ProjectsRef` | `lib/server/mongo/types/userProjects.ts:L44-L68` |
| `session` | `SessionRef` | `lib/server/mongo/types/userSessions.ts:L5-L14` |

All models exported from: `lib/server/mongo/init.ts:L17-L31`

### Relationships
- **UserCredentials ↔ UserDocument**: Share the same `_id` (UUID)
- **UserDocument → Projects**: Denormalized `projects[]` array with summaries; full Project docs linked by `id`
- **Project → Posts**: Posts embedded as subdocument array within Project
- **Session → User**: `user_id` references `UserCredentials._id`

## Auth & Middleware

### Authentication (Lucia Auth)
- **Lucia init**: `lib/server/lucia/init.ts:L11-L24` — Session-cookie based, MongoDB adapter
- **MongoDB adapter**: Uses `sessions` and `credentials` collections (`lib/server/lucia/init.ts:L5-L9`)
- **Session validation**: `lib/server/lucia/functions/validate-request.ts:L8-L51` — Cached via React's `cache()`, reads session cookie, validates with Lucia, refreshes if fresh
- **Password hashing**: Argon2 with env salt
  - Encode: `lib/server/encoding/encodePassword.ts:L3-L9`
  - Verify: `lib/server/encoding/isPasswordValid.ts:L3-L4`
- **Protected routes**: `components/containers/ProtectedRoute.tsx:L5-L17` — Server component, validates session, redirects to `/signin`
- **Protected layout**: `app/app/layout.tsx:L1-L6` — Wraps all `/app/*` pages in `ProtectedRoute`

### Middleware
- **Rate limiting**: `middleware.ts:L8-L37` — Applies to `/api/*` routes only
  - Uses Upstash Redis sliding window: 30 requests per minute per IP
  - Config: `lib/server/redis/init.ts:L15-L18`
  - Matcher: `middleware.ts:L40-L42` — `/api/:path*`

### Authorization Pattern
Each API route manually checks ownership:
1. Validate session via `validateRequest()`
2. Find the resource (project/post)
3. Compare `project.creator_uid === user.id`
4. Throw "Not authorized" if mismatch

### Helper Functions
- `validateRequestFetchUser`: `lib/server/utils/validateRequestFetchUser.ts:L5-L14` — Combines session validation + user document fetch
- `fetchUser` (server): `lib/server/functions/user/fetchUser.ts:L7-L21` — Internal API call with forwarded cookies
- `getProject` (server): `lib/server/functions/projects/getProject.ts` — Internal API call with forwarded cookies

## Configuration

| Variable | Purpose | Default | File |
|----------|---------|---------|------|
| `MONGO_DB_URI` | MongoDB connection string | None (required) | `lib/contants.tsx:L36` |
| `HASH_SALT` | Salt appended to passwords before Argon2 hashing | None (required) | `lib/server/encoding/encodePassword.ts:L4` |
| `ATOM_PROJECT_KEY` | Project key for the app's own blog (SDK usage) | None | `app/blog/page.tsx`, `app/sitemap.ts` |
| `UPSTASH_REDIS_REST_URL` | Upstash Redis URL for rate limiting | None (required) | `lib/server/redis/init.ts:L4` |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis auth token | None (required) | `lib/server/redis/init.ts:L5` |
| `NEXT_PUBLIC_ENV` | Environment flag (`"dev"` or `"prod"`) | `"dev"` → localhost:3000 | `lib/contants.tsx:L31-L34` |
| `NODE_ENV` | Node environment (controls secure cookies) | — | `lib/server/lucia/init.ts:L15` |

### Config Files
| File | Purpose |
|------|---------|
| `next.config.mjs` | Cache-Control headers (no-store) for `/`, `/app/*`, `/api/*` |
| `tailwind.config.ts` | shadcn/ui theme, typography plugin, color variables |
| `components.json` | shadcn/ui configuration |
| `tsconfig.json` | TypeScript config with `@/` path alias |
| `postcss.config.js` | PostCSS with Tailwind plugin |
| `.eslintrc.json` | ESLint with next/core-web-vitals |
| `packages/atom-nextjs/tsconfig.json` | SDK TypeScript config |

## Tests
- **No tests exist** in the main app. No test files, no test framework configured.
- The SDK package has `tsdx test --passWithNoTests` in its scripts, but no actual test files.
- CI workflow (`packages/atom-nextjs/.github/workflows/main.yml`) runs `yarn test --ci --coverage` but passes with no tests.

## Bruno API Collection
Manual API testing collection at `bruno/`:
- `bruno.json` — Collection config
- `bruno/environments/Development.bru` — Base URL (`localhost:3000`), secret vars for `session_id`, `project_key`, `project_id`, `post_id`
- `bruno/Routes/` — 7 route files for testing Create/Delete Post, Create/Delete Project, Get Post, Get Project, Get User

## Plan Limits
| Plan | Price | Max Projects | Max Posts | Max Body Length | Active |
|------|-------|-------------|-----------|-----------------|--------|
| `single` | Free | 2 | 100 | 10,000 chars | Yes |
| `startup` | $3.99/mo | 3 | 1,000 | 100,000 chars | No (disabled) |
| `business` | $11.99/mo | 5 | 2,500 | 500,000 chars | No (disabled) |

Defined in `lib/contants.tsx:L38-L87`. Free plan posts get a watermark appended (`app/api/posts/get/single/route.ts:L38-L40`).

## Open Questions
See `open-questions.md` for unresolved items.
