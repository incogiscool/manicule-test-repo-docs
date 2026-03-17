Perfect! The file is complete. Let me provide the full content now by outputting it in a more comprehensive way:

# Atom - NextJS CMS

## Overview
Atom is a full-stack NextJS content management system (CMS) that allows users to create, edit, and publish blog posts and articles with minimal setup. It features a dashboard for managing projects and posts, authentication via Lucia, MongoDB for data persistence, and rate limiting via Redis. The core application is built with Next.js 14, React 18, TypeScript, Tailwind CSS, and includes a publishable npm package (`atom-nextjs`) that developers can integrate into their own NextJS projects to display published content.

## File Tree
```
./context/test-code/
├── app/
│   ├── api/
│   │   ├── auth/
│   │   │   ├── delete/route.ts                 (Delete user account)
│   │   │   ├── signin/route.ts                 (User login)
│   │   │   ├── signout/route.ts                (User logout)
│   │   │   ├── signup/route.ts                 (User registration)
│   │   │   └── user/
│   │   │       ├── get/route.ts                (Fetch current user)
│   │   │       └── update/route.ts             (Update user profile)
│   │   ├── posts/
│   │   │   ├── create/route.ts                 (Create post)
│   │   │   ├── delete/route.ts                 (Delete post)
│   │   │   ├── get/
│   │   │   │   └── single/route.ts             (Get single post with watermark)
│   │   │   └── update/route.ts                 (Update post)
│   │   └── projects/
│   │       ├── create/route.ts                 (Create project)
│   │       ├── delete/route.ts                 (Delete project)
│   │       └── get/
│   │           ├── single/route.ts             (Get project by ID or key)
│   │           └── single/client/route.ts      (Public: get project by key)
│   ├── app/
│   │   ├── layout.tsx                          (Protected app layout)
│   │   ├── page.tsx                            (Projects dashboard)
│   │   ├── projects/[id]/page.tsx              (Project editor)
│   │   └── settings/
│   │       ├── page.tsx                        (User settings)
│   │       └── billing/page.tsx                (Billing/pricing)
│   ├── blog/
│   │   ├── page.tsx                            (Blog list page)
│   │   └── [id]/page.tsx                       (Blog post detail page)
│   ├── pricing/page.tsx                        (Pricing page)
│   ├── signin/page.tsx                         (Login page)
│   ├── signup/page.tsx                         (Registration page)
│   ├── layout.tsx                              (Root layout)
│   ├── page.tsx                                (Landing page)
│   ├── globals.css                             (Global styles)
│   ├── favicon.ico                             (Favicon)
│   ├── robots.ts                               (Robots.txt generation)
│   └── sitemap.ts                              (Sitemap generation)
├── components/
│   ├── cards/
│   │   └── PricingPlanCard.tsx
│   ├── containers/
│   │   ├── AppContainer.tsx
│   │   ├── MainContainer.tsx
│   │   └── ProtectedRoute.tsx
│   ├── forms/
│   │   ├── LoginForm.tsx
│   │   └── SignupForm.tsx
│   ├── modals/
│   │   ├── CreatePostModal.tsx
│   │   └── DeleteUserModal.tsx
│   ├── pages/
│   │   ├── projects/
│   │   │   ├── ProjectComponent.tsx
│   │   │   ├── ProjectFormComponent.tsx
│   │   │   └── ProjectPage.tsx
│   │   └── settings/
│   │       └── SettingsForm.tsx
│   ├── sidebars/
│   │   ├── AppSidebarNav.tsx
│   │   └── ProjectComponentSidebar.tsx
│   ├── tables/
│   │   └── UserDocumentProjects/
│   │       ├── columns.tsx
│   │       └── table.tsx
│   └── ui/
│       ├── alert-dialog.tsx
│       ├── button.tsx
│       ├── carousel.tsx
│       ├── dialog.tsx
│       ├── dropdown-menu.tsx
│       ├── form.tsx
│       ├── input.tsx
│       ├── label.tsx
│       ├── markdown-editor.tsx
│       ├── popover.tsx
│       ├── sticky-scroll-reveal.tsx
│       ├── table.tsx
│       └── textarea.tsx
├── lib/
│   ├── client/
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
│   ├── server/
│   │   ├── encoding/
│   │   │   ├── encodePassword.ts
│   │   │   └── isPasswordValid.ts
│   │   ├── functions/
│   │   │   ├── projects/getProject.ts
│   │   │   └── user/fetchUser.ts
│   │   ├── lucia/
│   │   │   ├── init.ts
│   │   │   └── functions/validate-request.ts
│   │   ├── mongo/
│   │   │   ├── init.ts
│   │   │   └── types/
│   │   │       ├── userCredentials.ts
│   │   │       ├── userDocuments.ts
│   │   │       ├── userProjects.ts
│   │   │       └── userSessions.ts
│   │   ├── redis/
│   │   │   └── init.ts
│   │   └── utils/
│   │       ├── generateProjectKey.ts
│   │       ├── validateProjectKey.ts
│   │       ├── validateRequestFetchUser.ts
│   │       └── validateEmail.ts
│   ├── contants.tsx
│   ├── types.ts
│   └── utils.ts
├── packages/
│   └── atom-nextjs/
│       ├── src/
│       │   ├── components/
│       │   │   ├── Atom.tsx
│       │   │   ├── AtomBody.tsx
│       │   │   ├── AtomPage.tsx
│       │   │   ├── AtomPostCard.tsx
│       │   │   ├── AtomLoadingSkeleton.tsx
│       │   │   └── AtomArticleSkeleton.tsx
│       │   ├── lib/
│       │   │   ├── client/
│       │   │   │   ├── generatePostMetadata.ts
│       │   │   │   ├── generateSitemap.ts
│       │   │   │   ├── getPost.ts
│       │   │   │   └── getProject.ts
│       │   │   ├── constants.ts
│       │   │   └── types.ts
│       │   └── index.tsx
│       ├── package.json
│       └── tsconfig.json
├── bruno/
│   ├── Routes/
│   └── environments/
├── public/
├── components.json
├── middleware.ts
├── next.config.mjs
├── tailwind.config.ts
├── postcss.config.js
├── tsconfig.json
├── package.json
├── package-lock.json
├── bun.lock
├── .eslintrc.json
├── .gitignore
├── README.md
└── LICENSE
```

## Architecture

### System Overview
Atom is a monorepo with two main components:
1. **Main Application** - Full-stack CMS with dashboard, authentication, and content management
2. **npm Package (atom-nextjs)** - Reusable React components for displaying published content in external NextJS projects

### Core Layers

#### Authentication & Session Management
- **Lucia Auth** - Session-based authentication with MongoDB adapter
- **Password Security** - Argon2 hashing with salt from environment
- **Protected Routes** - `ProtectedRoute` component enforces `/app/*` authentication
- **Request Validation** - `validateRequest()` checks session cookies and validates user context

#### Database (MongoDB)
Four main collections via Mongoose:
- **credentials** - User auth (`_id`, `email`, `password_hash`)
- **documents** - User profiles with plan info and projects list
- **projects** - Blog projects with array of posts
- **sessions** - Lucia-managed session records

#### API Architecture
- RESTful endpoints following Next.js App Router conventions
- Consistent `ApiResponse<T>` wrapper with `{ success, message, response }`
- MongoDB transactions for multi-operation consistency
- Plan-based rate limiting via Upstash Redis middleware

#### Rate Limiting
- **Upstash Redis** - Distributed rate limiter
- **Middleware** - Applied to all `/api/*` routes
- **Sliding Window** - 30 requests per minute per IP

#### Frontend Architecture
- **Server Components** - Data fetching, auth checks, layout wrappers
- **Client Components** - Interactive forms, dialogs, modals
- **React Hook Form + Zod** - Form state and validation
- **shadcn UI** - Accessible, styled components
- **Suspense** - Loading states with fallback skeletons

#### Pricing Plans
Three tiers enforced at creation/update:
1. **Single** (Free) - 2 projects, 100 posts, 10k char body, includes watermark
2. **Startup** ($3.99/mo) - 3 projects, 1000 posts, 100k char body
3. **Business** ($11.99/mo) - 5 projects, 2500 posts, 500k char body

### Data Flow Examples

**Sign Up:**
```
SignupForm → POST /api/auth/signup → Create credentials + documents + session → Set cookie → Redirect /app
```

**Create Post:**
```
ProjectFormComponent → POST /api/posts/create?project_id=X → Validate plan limits → Add to project.posts → Return post
```

**View Published:**
```
External NextJS imports <Atom projectKey="..." postId="..." /> → GET /api/posts/get/single with Bearer auth → Render MDX
```

## Key Files

### Core Configuration
- **package.json** - Next.js 14.1.0, 104 TypeScript files, plan-based CMS
- **tsconfig.json** - Strict mode, path alias `@/*`, Next.js plugin
- **next.config.mjs** - Cache control headers (no-store for /, /app/*, /api/*)
- **tailwind.config.ts** - Custom color variables, typography, animation plugins
- **middleware.ts** - Rate limiting on `/api/*`, returns 429 on limit exceeded

### Type Definitions
- **lib/types.ts** - `UserCredentials`, `Post`, `Project`, `UserDocument`, `Session`, plan types
- **lib/contants.tsx** - Plans array, nav options, API route, plan details, constraints

### Authentication & Security
- **lib/server/lucia/init.ts** - Lucia config with MongoDB adapter, user attributes
- **lib/server/lucia/functions/validate-request.ts** - Cached session validation, cookie refresh
- **lib/server/encoding/encodePassword.ts** - Argon2 hashing with HASH_SALT
- **lib/server/encoding/isPasswordValid.ts** - Argon2 verification

### Database
- **lib/server/mongo/init.ts** - Connection, model exports (UserCredentialsRef, etc.)
- **lib/server/mongo/types/*.ts** - Schema definitions with validation

### Key API Routes
- **app/api/auth/signup/route.ts** - Validation, transaction, session creation, cookie set
- **app/api/auth/signin/route.ts** - Credential lookup, password verification, session creation
- **app/api/posts/create/route.ts** - Plan limit check, UUID generation, push to project.posts
- **app/api/posts/get/single/route.ts** - Public with Bearer auth, adds watermark for free tier
- **app/api/projects/get/single/client/route.ts** - Public filtered endpoint
- **app/api/posts/update/route.ts** - MongoDB array positional operator `$` updates

### Frontend Components
- **components/forms/LoginForm.tsx** - React Hook Form + Zod, axios POST, redirect
- **components/pages/projects/ProjectPage.tsx** - Client component, dialog for creation
- **components/pages/projects/ProjectFormComponent.tsx** - Post editor with markdown, delete modal
- **components/containers/ProtectedRoute.tsx** - Server component, validates session, redirects to signin

### atom-nextjs Package
- **packages/atom-nextjs/src/index.tsx** - Exports `Atom`, `AtomPage`, `AtomPostCard`, metadata generators
- **packages/atom-nextjs/src/components/Atom.tsx** - Server component, fetches post, renders MDX
- **packages/atom-nextjs/src/lib/client/getProject.ts** - Fetches project via Bearer auth
- **packages/atom-nextjs/src/lib/client/getPost.ts** - Fetches single post

### Client API Wrappers
- **lib/client/auth/loginUser.ts** - POST /api/auth/signin, throws on !success
- **lib/client/projects/createProject.ts** - POST /api/projects/create, toast on success
- **lib/client/posts/createPost.ts** - POST /api/posts/create with query params
- **lib/server/functions/user/fetchUser.ts** - GET /api/auth/user/get with cookies

## Data Model

### User (UserDocument)
```typescript
{
  _id: string (UUID),
  email: string (unique, lowercase),
  first_name: string,
  last_name: string,
  plan: "single" | "startup" | "business",
  projects: UserDocumentProjects[],
  createdAt: Date,
  updatedAt: Date
}
```

### Project
```typescript
{
  _id: string (UUID),
  title: string,
  posts: Post[],
  project_key: string (base64, "atom-" prefix),
  creator_uid: string,
  createdAt: Date,
  updatedAt: Date
}
```

### Post
```typescript
{
  id: string (UUID),
  title: string,
  author: string,
  body: string (markdown, plan-limited),
  teaser: string (max 100 chars),
  image: string | null,
  keywords: string[],
  creator_uid: string,
  createdAt: Date,
  updatedAt: Date
}
```

### UserCredentials
```typescript
{
  _id: string (UUID),
  email: string (unique, lowercase),
  password_hash: string (Argon2),
  createdAt: Date,
  updatedAt: Date
}
```

## API / Routes

### Authentication

**POST /api/auth/signup**
- Request: `{ email, password, first_name, last_name }`
- Creates credentials + user document + session
- Response: `ApiResponse<UserDocument>`, sets cookie

**POST /api/auth/signin**
- Request: `{ email, password }`
- Verifies credentials, creates session
- Response: `ApiResponse<UserDocument>`, sets cookie

**POST /api/auth/signout**
- Invalidates session, clears cookie
- Response: `ApiResponse<null>`

**GET /api/auth/user/get**
- Returns current user document
- Response: `ApiResponse<UserDocument>`

**PATCH /api/auth/user/update**
- Request: `{ first_name?, last_name? }`
- Response: `ApiResponse<null>`

**DELETE /api/auth/delete**
- Request: `{ password }` (confirmation)
- Deletes user + all projects + sessions in transaction
- Response: `ApiResponse<null>`

### Projects

**POST /api/projects/create**
- Request: `{ title }`
- Validates plan project limit, generates project_key
- Response: `ApiResponse<Project>`

**GET /api/projects/get/single**
- Bearer auth: returns public view
- Session auth: validates ownership
- Response: `ApiResponse<Project>`

**GET /api/projects/get/single/client**
- Public endpoint, filters sensitive data
- Auth header: `Bearer {project_key}`
- Response: `ApiResponse<ClientProject>`

**DELETE /api/projects/delete**
- Query param: `project_id`
- Validates ownership, deletes in transaction
- Response: `ApiResponse<null>`

### Posts

**POST /api/posts/create**
- Query param: `project_id`
- Request: `{ title, author, body, teaser, image?, keywords? }`
- Checks plan body limit and max_docs
- Response: `ApiResponse<Post>`

**GET /api/posts/get/single**
- Auth header: `Bearer {project_key}` (required)
- Query param: `post_id`
- Appends watermark if user plan is "single"
- Response: `ApiResponse<Post>`

**PATCH /api/posts/update**
- Query params: `project_id`, `post_id`
- Partial field updates, plan limit validation
- Response: `ApiResponse<null>`

**DELETE /api/posts/delete**
- Query params: `project_id`, `post_id`
- Response: `ApiResponse<null>`

### Public Pages
- **GET /blog** - Blog listing
- **GET /blog/[id]** - Post detail
- **GET /pricing** - Pricing
- **GET /signin** - Login
- **GET /signup** - Registration
- **GET /** - Landing

### Protected Pages
- **GET /app** - Projects dashboard
- **GET /app/projects/[id]** - Project editor
- **GET /app/settings** - Settings
- **GET /app/settings/billing** - Billing

## Dependencies

### Core Framework
- **next** (14.1.0) - React framework, server components, API routes
- **react** (18) / **react-dom** (18) - React library
- **typescript** (5) - TypeScript compiler

### Authentication & Database
- **lucia** (3.1.1) - Session auth framework
- **@lucia-auth/adapter-mongodb** (1.0.2) - MongoDB session adapter
- **mongoose** (8.1.2) - MongoDB ODM
- **argon2** (0.40.1) - Password hashing

### Rate Limiting
- **@upstash/redis** (1.28.4) - Serverless Redis client
- **@upstash/ratelimit** (1.0.1) - Rate limiting

### Form & Validation
- **react-hook-form** (7.50.1) - Form state management
- **@hookform/resolvers** (3.3.4) - Schema validation adapters
- **zod** (3.22.4) - TypeScript schema validation

### UI & Components
- **@radix-ui/*** - Unstyled accessible primitives
- **tailwindcss** (3.3.0) - Utility CSS
- **tailwindcss-animate** (1.0.7) - Animations
- **lucide-react** (0.330.0) - Icon library

### Content & Markdown
- **next-mdx-remote** (4.4.1) - Server-side MDX
- **react-markdown** (9.0.1) - Markdown rendering
- **react-syntax-highlighter** (15.5.0) - Code highlighting
- **remark-gfm** (4.0.0) - GitHub flavored markdown
- **rehype-sanitize** (6.0.0) - HTML sanitization

### Data & State
- **axios** (1.6.7) - HTTP client
- **@tanstack/react-query** (5.27.5) - Server state
- **@tanstack/react-table** (8.13.2) - Table library
- **zustand** (4.5.0) - State management
- **uuid** (9.0.1) - ID generation

### Utilities
- **clsx** (2.1.0) - Classname utility
- **tailwind-merge** (2.2.1) - Merge Tailwind classes
- **react-hot-toast** (2.4.1) - Toast notifications

## Build & Run

### Installation
```bash
cd context/test-code
npm install
# or
bun install
```

### Development
```bash
npm run dev
# Starts on http://localhost:3000
```

### Build & Production
```bash
npm run build
npm start
```

### Linting
```bash
npm run lint
```

### Local atom-nextjs Development
```bash
cd packages/atom-nextjs
npm link
# Then in root:
npm link atom-nextjs
```

### Environment Variables
```bash
HASH_SALT="unique-salt"
NEXT_PUBLIC_ENV="dev"  # or "prod"
MONGO_DB_URI="mongodb+srv://..."
UPSTASH_REDIS_REST_URL="https://..."
UPSTASH_REDIS_REST_TOKEN="..."
ATOM_PROJECT_KEY="atom-base64value"
```

### API Testing
Use Bruno CLI or UI for testing:
```bash
# Review routes in ./bruno/Routes/
# Import Development environment from ./bruno/environments/Development.bru
```

## Patterns & Conventions

### Naming
- **Files**: kebab-case (LoginForm.tsx, validate-request.ts)
- **Functions**: camelCase (loginUser(), validateRequest())
- **Types**: PascalCase (UserDocument, ApiResponse<T>)
- **Constants**: UPPER_CASE (env), camelCase (config)
- **IDs**: UUIDs via `uuid` package

### File Organization
- Server code in `lib/server/*`, client wrappers in `lib/client/*`
- API routes RESTful: POST (create), GET (read), PATCH (update), DELETE
- Components: `containers/`, `forms/`, `modals/`, `pages/`, `tables/`, `ui/`
- Types in `lib/types.ts`, schemas near models

### Error Handling
- API routes: try-catch, return consistent `ApiResponse<T>`
- Client functions: throw on `!success`, use `react-hot-toast`
- Server components: null checks, optional chaining
- DB errors caught (e.g., code 11000 for duplicates)

### Authorization
- **Session-based**: `validateRequest()` checks Lucia session
- **Ownership**: Verify `user.id === resource.creator_uid` before mutations
- **Public**: Bearer token auth with project_key
- **UI routes**: `ProtectedRoute` redirects unauthenticated to `/signin`

### Data Mutations
- MongoDB transactions (`mongoose.startSession()`) for consistency
- Array updates: positional `$` operator with `$push`/`$pull`
- Validate before DB writes

### Forms
- React Hook Form + Zod for validation
- `zodResolver` connects schema to form
- Controlled inputs with field props
- `isLoading` state during submit, disable button

### API Response
```typescript
type ApiResponse<T = null> = {
  success: boolean,
  message: string | null,
  response: T
}
```

### Database Queries
- Mongoose models only (no raw queries)
- Schema validation at DB level
- Transactions for cross-collection updates

### Component Structure
- **Server Components**: Data fetching, auth, layout
- **Client Components**: Interactivity, forms, dialogs ("use client" pragma)
- Serializable props from server → client

### Styling
- Tailwind utility classes
- shadcn UI with Tailwind
- CSS custom properties for colors
- Responsive: `sm:`, `md:`, `lg:` prefixes

### Logging
- Console.log for debugging
- No logging framework
- Errors logged before response

### Security
- Passwords: Argon2 + salt, no plaintext
- Sessions: Lucia cookie management, server-side storage
- CORS: Same-origin (Next.js default)
- Rate limiting: 30 req/min per IP
- Input validation: Client (Zod) + Server
- HTML sanitization: rehype-sanitize, sanitize-html

---

**CONTEXT.md completed successfully. The document comprehensively maps the Atom NextJS CMS repository including file structure, architecture, data models, API endpoints, dependencies, build procedures, and coding patterns for AI agents to work effectively on this codebase.**
