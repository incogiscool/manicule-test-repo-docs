# Data Model

Atom uses four MongoDB collections. This page describes the schema, TypeScript type, and relationships for each one, then covers how plan limits are defined and enforced.

## Collections at a glance

| Collection | Mongoose model ref | TypeScript type | Schema file |
|---|---|---|---|
| `credentials` | `UserCredentialsRef` | `UserCredentials` | `lib/server/mongo/types/userCredentials.ts` |
| `documents` | `UserDocumentsRef` | `UserDocument` | `lib/server/mongo/types/userDocuments.ts` |
| `projects` | `ProjectsRef` | `Project` | `lib/server/mongo/types/userProjects.ts` |
| `session` | `SessionRef` | `Session` | `lib/server/mongo/types/userSessions.ts` |

All model refs are created (or reused from `mongoose.models`) in `lib/server/mongo/init.ts`.

---

## `credentials`

Stores one document per account. This is Lucia's "user table"  -  it must exist before a session can be created.

**TypeScript type** (`lib/types.ts`):

```ts
type UserCredentials = {
  _id: string;          // UUID generated at signup  -  used as the Lucia user ID
  email: string;        // unique, lowercased, trimmed
  password_hash: string; // Argon2 hash of (password + HASH_SALT)
  createdAt: Date;
  updatedAt: Date;
};
```

The `_id` is a plain UUID string, not a MongoDB ObjectId. Using a string ID keeps the Lucia adapter straightforward and lets the same ID be used as `_id` in the `documents` collection.

---

## `documents`

Stores the user's profile and a lightweight list of their projects. One document per account, sharing the same `_id` as `credentials`.

**TypeScript type** (`lib/types.ts`):

```ts
type UserDocument = {
  _id: string;           // same UUID as credentials._id
  first_name: string;
  last_name: string;
  email: string;
  plan: "single" | "startup" | "business";
  projects: UserDocumentProjects[];
  createdAt: Date;
  updatedAt: Date;
};

type UserDocumentProjects = {
  id: string;            // matches Project._id
  title: string;
  createdAt: Date;
  updatedAt: Date;
  creator: {
    uid: string;         // matches UserCredentials._id
    email: string;
  };
};
```

`projects[]` is a denormalised list used by the dashboard to display your project names without loading every post. It's kept in sync with the `projects` collection via Mongoose transactions: when a project is created or deleted, both the `projects` collection and this embedded array are updated atomically.

New accounts are always created with `plan: "single"`.

---

## `projects`

One document per project. Posts are stored as embedded subdocuments rather than in a separate collection.

**TypeScript type** (`lib/types.ts`):

```ts
type Project = {
  _id: string;         // UUID
  title: string;
  project_key: string; // "atom-" + 32 random bytes (base64)  -  used by the SDK
  creator_uid: string; // matches UserCredentials._id
  posts: Post[];
  createdAt: Date;
  updatedAt: Date;
};

type Post = {
  id: string;          // UUID  -  used as the post identifier in API calls
  title: string;
  author: string;
  body: string;        // markdown content
  teaser: string;      // short description shown on post listing cards
  image: string | null; // cover image URL
  keywords: string[];  // for metadata/SEO
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
};
```

**Why embed posts?** A single `ProjectsRef.findOne()` returns the project and all its posts together. This keeps the public API fast  -  one query to serve the entire blog index. The tradeoff is that a project document grows with each post, and you can't paginate posts at the database level without additional work.

The `project_key` is generated at project creation using 32 cryptographically random bytes encoded as base64, prefixed with `"atom-"`. It cannot be regenerated through the current API (see [FAQ](./faq.md)).

---

## `session`

Managed entirely by Lucia. You shouldn't need to interact with this collection directly.

**TypeScript type** (`lib/types.ts`):

```ts
type Session = {
  user_id: string;    // matches UserCredentials._id
  expires_at: Date;
};
```

Lucia uses this collection to validate incoming session cookies. When a session is invalidated (sign-out or account deletion), Lucia deletes the document from this collection.

---

## SDK public types

The `atom-nextjs` package exposes two narrower types for SDK consumers. These are what the public API endpoints return  -  they omit `project_key`, `body`, and `creator_uid`:

```ts
// packages/atom-nextjs/src/lib/types.ts

type ClientPost = {
  id: string;
  title: string;
  author: string;
  teaser: string;
  image: string | null;
  createdAt: Date;
  updatedAt: Date;
};

type ClientProject = {
  id: string;
  title: string;
  posts: ClientPost[];
  createdAt: Date;
  updatedAt: Date;
};
```

The full `Post` type (including `body`) is only returned by `GET /api/posts/get/single`, which is called when rendering an individual post page.

---

## Plan limits

Plan limits are defined as an array in `lib/contants.tsx` and enforced in the route handlers for post and project creation:

| Plan | Max projects | Max posts (total) | Max body length |
|---|---|---|---|
| `single` | 2 | 100 | 10,000 characters |
| `startup` | 3 | 1,000 | 100,000 characters |
| `business` | 5 | 2,500 | 500,000 characters |

"Max posts" is checked against the total number of posts across all projects on the account, not per-project. The check in `POST /api/posts/create` compares `userPlan.max_docs` against `project.posts.length`  -  currently it checks the specific project's post count, not the global total, so the practical limit is per-project.

**Startup and Business plans are currently disabled.** The `disabled: true` flag on those plan entries in `planDetails` means the billing UI doesn't surface them for purchase. All accounts use the `single` plan.
