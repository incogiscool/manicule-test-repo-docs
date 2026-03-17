# Data Model

This page documents Atom's MongoDB collections, Mongoose schemas, TypeScript types, how entities relate to each other, and the plan-based resource limits.

## Collections overview

Atom uses four MongoDB collections:

| Collection | Mongoose model | What it stores |
|---|---|---|
| `credentials` | `UserCredentialsRef` | Login credentials (email + password hash) |
| `documents` | `UserDocumentsRef` | User profiles (name, email, plan, project summaries) |
| `projects` | `ProjectsRef` | Full project documents with embedded posts |
| `sessions` | `SessionRef` | Lucia auth sessions |

All models are exported from `lib/server/mongo/init.ts`.

## Entity relationships

```
UserCredentials ←──── same _id (UUID) ────→ UserDocument
       ↑                                         |
       |                                    projects[] (summaries)
  Session.user_id                                 |
                                                  ↓
                                      Project (full document)
                                           |
                                      posts[] (embedded subdocuments)
                                           |
                                           ↓
                                         Post
```

The key relationships:

- **UserCredentials and UserDocument share the same `_id`**. When a user signs up, both documents are created in a transaction with the same UUID.
- **UserDocument contains a denormalized `projects` array** with summaries (title, ID, timestamps, creator). The full project data lives in the `projects` collection.
- **Posts are embedded inside Project documents** as a subdocument array, not stored in a separate collection.
- **Session references UserCredentials** via `user_id`, which equals the shared `_id`.

## credentials collection

Stores login credentials. One document per user.

**TypeScript type** (`lib/types.ts`):

```typescript
type UserCredentials = {
  email: string;
  password_hash: string;
  createdAt: Date;
  updatedAt: Date;
  _id: string;
};
```

**Mongoose schema** (`lib/server/mongo/types/userCredentials.ts`):

| Field | Type | Required | Notes |
|---|---|---|---|
| `_id` | `String` | Yes | UUID, set manually (not auto-generated ObjectId) |
| `email` | `String` | Yes | Unique, lowercased, trimmed |
| `password_hash` | `String` | Yes | Argon2 hash of password + salt |
| `createdAt` | `Date` | Auto | From `timestamps: true` |
| `updatedAt` | `Date` | Auto | From `timestamps: true` |

The schema uses `{ timestamps: true, _id: false }` to auto-manage timestamps while using a custom string `_id` instead of MongoDB's default ObjectId.

The `email` field has a unique index, which is how duplicate email signups are caught (MongoDB error code 11000).

## documents collection

Stores user profiles and denormalized project summaries. One document per user.

**TypeScript type** (`lib/types.ts`):

```typescript
type UserDocument = {
  _id: string;
  first_name: string;
  last_name: string;
  createdAt: Date;
  updatedAt: Date;
  projects: UserDocumentProjects[];
  email: string;
  plan: Plan;
};

type Plan = "single" | "startup" | "business";

type UserDocumentProjects = {
  id: string;
  title: string;
  createdAt: Date;
  updatedAt: Date;
  creator: UserDocumnetProjectsCreator;
};

type UserDocumnetProjectsCreator = {
  uid: string;
  email: string;
};
```

**Mongoose schema** (`lib/server/mongo/types/userDocuments.ts`):

| Field | Type | Required | Notes |
|---|---|---|---|
| `_id` | `String` | Yes | Same UUID as the matching `credentials` document |
| `first_name` | `String` | Yes | |
| `last_name` | `String` | Yes | |
| `email` | `String` | Yes | |
| `plan` | `String` (enum) | Yes | One of `"single"`, `"startup"`, `"business"` |
| `projects` | Array of subdocuments | Yes | Denormalized project summaries (see below) |
| `createdAt` | `Date` | Auto | |
| `updatedAt` | `Date` | Auto | |

Each entry in the `projects` array has this shape:

| Field | Type | Notes |
|---|---|---|
| `id` | `String` | The project's `_id` in the `projects` collection |
| `title` | `String` | Project title |
| `createdAt` | `Date` | Copied from the project document |
| `updatedAt` | `Date` | Copied from the project document |
| `creator.uid` | `String` | The user's `_id` |
| `creator.email` | `String` | The user's email |

This denormalization means the dashboard can display a project list without querying the `projects` collection. The tradeoff: if a project title changes, both the `projects` collection and this embedded summary need updating.

## projects collection

Stores full project documents with posts embedded as subdocuments.

**TypeScript type** (`lib/types.ts`):

```typescript
type Project = {
  title: string;
  _id: string;
  posts: Post[];
  project_key: string;
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
};
```

**Mongoose schema** (`lib/server/mongo/types/userProjects.ts`):

| Field | Type | Required | Notes |
|---|---|---|---|
| `_id` | `String` | Yes | UUID |
| `title` | `String` | Yes | Max 50 characters (enforced in API, not schema) |
| `posts` | Array of Post subdocuments | Yes | See Post schema below |
| `project_key` | `String` | Yes | `atom-` + 32 random bytes in base64 |
| `creator_uid` | `String` | Yes | References `credentials._id` |
| `createdAt` | `Date` | Auto | |
| `updatedAt` | `Date` | Auto | |

The `project_key` is generated by `lib/server/utils/generateProjectKey.ts`:

```typescript
import { randomBytes } from "crypto";

export const generateProjectKey = () => {
  const size = 32;
  const format = "base64";
  const buffer = randomBytes(size);
  return "atom-" + buffer.toString(format);
};
```

### Post subdocument

Posts are embedded directly in the project's `posts` array.

**TypeScript type** (`lib/types.ts`):

```typescript
type Post = {
  createdAt: Date;
  id: string;
  updatedAt: Date;
  title: string;
  author: string;
  body: string;
  image: string | null;
  creator_uid: string;
  keywords?: string[];
  teaser: string;
};
```

**Mongoose schema** (`lib/server/mongo/types/userProjects.ts`):

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | `String` | Yes | UUID (note: `id`, not `_id`) |
| `title` | `String` | Yes | |
| `author` | `String` | Yes | Display name |
| `body` | `String` | Yes | MDX content |
| `image` | `String` | No | Cover image URL |
| `creator_uid` | `String` | Yes | References `credentials._id` |
| `keywords` | `[String]` | No | Array of tags |
| `teaser` | `String` | Yes | Short summary |
| `createdAt` | `Date` | Auto | |
| `updatedAt` | `Date` | Auto | |

Posts use `{ timestamps: true, _id: false }`. They have a custom `id` field (a UUID string) instead of MongoDB's default `_id`. This `id` is what the SDK uses to fetch individual posts.

## sessions collection

Stores Lucia auth sessions. Managed entirely by Lucia, not by application code directly.

**TypeScript type** (`lib/types.ts`):

```typescript
type Session = {
  user_id: string;
  expires_at: Date;
};
```

**Mongoose schema** (`lib/server/mongo/types/userSessions.ts`):

| Field | Type | Required | Notes |
|---|---|---|---|
| `_id` | ObjectId | Auto | Default MongoDB ObjectId (managed by Lucia) |
| `user_id` | `String` | Yes | References `credentials._id` |
| `expires_at` | `Date` | Yes | Session expiration timestamp |

Unlike other collections, sessions use MongoDB's default `_id` (ObjectId) because Lucia manages the session lifecycle.

## SDK types

The SDK (`packages/atom-nextjs/`) defines its own sanitized types for public API responses (`packages/atom-nextjs/src/lib/types.ts`):

```typescript
type ClientPost = {
  image?: string | null;
  id: string;
  teaser: string;
  title: string;
  createdAt: Date;
  updatedAt: Date;
  author: string;
};

type ClientProject = {
  title: string;
  posts: ClientPost[];
  id: string;
  updatedAt: Date;
  createdAt: Date;
};

type ApiResponse<T = null> = {
  response: T;
  success: boolean;
  message: string;
};
```

`ClientPost` omits `body`, `keywords`, and `creator_uid` compared to the full `Post` type. `ClientProject` omits `project_key` and `creator_uid` compared to the full `Project` type. This sanitization happens at the API layer in `/api/projects/get/single/client`.

## Plan limits

Resource limits are defined in `lib/contants.tsx`. Each plan specifies maximum projects, total posts, and body character length:

| Plan | Price | Max projects | Max posts | Max body length |
|---|---|---|---|---|
| **Single** | Free | 2 | 100 | 10,000 characters |
| **Startup** | $3.99/mo | 3 | 1,000 | 100,000 characters |
| **Business** | $11.99/mo | 5 | 2,500 | 500,000 characters |

Currently, only the Single plan is active. The Startup and Business plans are defined in the code with `disabled: true`.

The plan configuration type:

```typescript
type PlanDetailsPlan = {
  title: string;
  id: Plan;
  price: number | null;
  description: string;
  max_docs: number;
  max_body_length: number;
  features: string[];
  max_projects: number;
  active: boolean;
  disabled: boolean;
};
```

Plan limits are enforced in the API route handlers. For example, when creating a post, the server checks the user's plan against `max_body_length`. When creating a project, it checks `max_projects` against the current count.

Posts on the Single plan also receive a Markdown watermark appended to their body when fetched through the public API: `"This post was created using [Atom](https://atomcms.vercel.app/)"`.
