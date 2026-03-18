# API Reference

All routes are under the base path `/api`. In production the base URL is `https://cmsatom.netlify.app`; locally it's `http://localhost:3000`.

Every response is wrapped in the same envelope:

```ts
type ApiResponse<T = null> = {
  success: boolean;      // true on success, false on any error
  message: string | null; // human-readable result or error description
  response: T;           // the actual payload, or null on error
};
```

**Rate limiting:** Every `/api/*` request is limited to 30 requests per minute per IP using a sliding window. Requests over the limit receive a 200 response with `success: false` and `message: "Too many requests."` (the rate limiter runs in middleware before the route handler, so no HTTP 429 is used).

---

## Authentication

### POST `/api/auth/signup`

Creates a new account and opens a session.

**Auth required:** No

**Request body:**

```ts
{
  email: string;       // must be a valid email address
  password: string;    // minimum 8 characters
  first_name: string;  // max 30 characters
  last_name: string;   // max 30 characters
}
```

**Response `200`:**

```ts
ApiResponse<UserDocument>
// response.plan is always "single" for new accounts
```

**Errors:**

| Condition | `message` |
|---|---|
| Email already registered | `"Email already in use."` |
| Email invalid | `"Email not valid."` |
| Password too short | `"Password must be at least 8 characters long."` |
| First/last name missing or too long | `"Invalid first name."` / `"Invalid last name."` |

On success a Lucia session cookie is set automatically. Subsequent requests to authenticated endpoints will be recognized by this cookie.

---

### POST `/api/auth/signin`

Authenticates an existing account and opens a session.

**Auth required:** No

**Request body:**

```ts
{
  email: string;
  password: string;
}
```

**Response `200`:**

```ts
ApiResponse<UserDocument>
```

A session cookie is set on success. On failure (wrong password or unknown email), `success` is `false` and no cookie is set.

---

### POST `/api/auth/signout`

Invalidates the current session.

**Auth required:** Yes (session cookie)

**Request body:** None

**Response `200`:**

```ts
ApiResponse<null>
// { success: true, message: "Successfuly signed out user.", response: null }
```

The session cookie is cleared in the response.

---

### DELETE `/api/auth/delete`

Permanently deletes the account, all of its projects, all posts, and all sessions. This action is irreversible.

**Auth required:** Yes (session cookie + password confirmation)

**Request body:**

```ts
{
  password: string;  // current account password, required for confirmation
}
```

**Response `200`:**

```ts
ApiResponse<null>
// { success: true, message: "Successfuly deleted user.", response: null }
```

The deletion runs inside a Mongoose transaction, so if any step fails the entire operation is rolled back.

---

### GET `/api/auth/user/get`

Returns the current user's document.

**Auth required:** Yes (session cookie)

**Query params:** None

**Response `200`:**

```ts
ApiResponse<UserDocument>
```

`UserDocument` shape:

```ts
{
  _id: string;
  first_name: string;
  last_name: string;
  email: string;
  plan: "single" | "startup" | "business";
  projects: Array<{
    id: string;
    title: string;
    createdAt: Date;
    updatedAt: Date;
    creator: { uid: string; email: string };
  }>;
  createdAt: Date;
  updatedAt: Date;
}
```

---

### PATCH `/api/auth/user/update`

Updates the current user's name.

**Auth required:** Yes (session cookie)

**Request body** (all fields optional; only provided fields are updated):

```ts
{
  first_name?: string;  // max 30 characters
  last_name?: string;   // max 30 characters
}
```

**Response `200`:**

```ts
ApiResponse<null>
// { success: true, message: "Successfuly updated user.", response: null }
```

---

## Projects

### POST `/api/projects/create`

Creates a new project and a generated `project_key`.

**Auth required:** Yes (session cookie)

**Request body:**

```ts
{
  title: string;  // max 50 characters, required
}
```

**Response `200`:**

```ts
ApiResponse<Project>
```

`Project` shape:

```ts
{
  _id: string;
  title: string;
  posts: Post[];
  project_key: string;  // "atom-" + 32 random bytes, base64-encoded
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
}
```

**Errors:**

| Condition | `message` |
|---|---|
| Plan project limit reached | `"Cannot create more than N projects."` |
| Title too long or empty | `"Invalid title."` |

The project limit comes from the user's active plan: `single` allows 2 projects, `startup` allows 3, `business` allows 5.

---

### DELETE `/api/projects/delete`

Deletes a project and all of its posts.

**Auth required:** Yes (session cookie)

**Query params:**

| Param | Required | Description |
|---|---|---|
| `project_id` | Yes | The `_id` of the project to delete |

**Response `200`:**

```ts
ApiResponse<null>
```

Only the project's owner can delete it. The operation runs in a transaction that also removes the project reference from the user's `UserDocument`.

---

### GET `/api/projects/get/single`

Returns a full `Project` document including `project_key`. Supports two authentication modes.

**Mode 1 — session (dashboard use):**

- **Auth:** Session cookie
- **Query params:** `?project_id=<id>`
- **Returns:** Full `Project` including `project_key`

**Mode 2 — Bearer token:**

- **Auth:** `Authorization: Bearer <project_key>` header
- **Query params:** None (the project is identified by the key)
- **Returns:** Full `Project` including `project_key`

**Response `200`:**

```ts
ApiResponse<Project>
```

---

### GET `/api/projects/get/single/client`

Public endpoint used by the `atom-nextjs` SDK. Returns a `ClientProject` with `project_key` omitted.

**Auth required:** `Authorization: Bearer <project_key>` header

**Query params:** None

**Response `200`:**

```ts
ApiResponse<ClientProject>
```

`ClientProject` shape:

```ts
{
  id: string;
  title: string;
  posts: ClientPost[];   // public fields only — no body, no creator_uid
  createdAt: Date;
  updatedAt: Date;
}
```

`ClientPost` shape:

```ts
{
  id: string;
  title: string;
  author: string;
  teaser: string;
  image: string | null;
  createdAt: Date;
  updatedAt: Date;
}
```

---

## Posts

### POST `/api/posts/create`

Creates a new post inside a project.

**Auth required:** Yes (session cookie)

**Query params:**

| Param | Required | Description |
|---|---|---|
| `project_id` | Yes | The `_id` of the project to add the post to |

**Request body:**

```ts
{
  title: string;       // required, max 50 characters
  author: string;      // required
  body: string;        // required; max length depends on plan (see below)
  teaser: string;      // required; brief description shown on post cards
  image?: string;      // optional URL for the cover image
  keywords?: string;   // optional, comma-separated list (e.g. "nextjs,cms,blog")
}
```

Body length limits by plan:
- `single`: 10,000 characters
- `startup`: 100,000 characters
- `business`: 500,000 characters

Keywords are split on the comma character into an array before storage.

**Response `200`:**

```ts
ApiResponse<Post>
```

`Post` shape:

```ts
{
  id: string;
  title: string;
  author: string;
  body: string;          // full markdown body
  teaser: string;
  image: string | null;
  keywords: string[];
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
}
```

---

### DELETE `/api/posts/delete`

Removes a post from its project.

**Auth required:** Yes (session cookie)

**Query params:**

| Param | Required | Description |
|---|---|---|
| `project_id` | Yes | The `_id` of the project containing the post |
| `post_id` | Yes | The `id` of the post to delete |

**Response `200`:**

```ts
ApiResponse<null>
```

---

### PATCH `/api/posts/update`

Updates one or more fields on an existing post. Only fields included in the request body are changed; omitted fields are left unchanged.

**Auth required:** Yes (session cookie)

**Query params:**

| Param | Required | Description |
|---|---|---|
| `project_id` | Yes | The `_id` of the project containing the post |
| `post_id` | Yes | The `id` of the post to update |

**Request body** (all fields optional):

```ts
{
  title?: string;     // max 50 characters
  author?: string;    // max 30 characters
  body?: string;      // max length enforced by plan
  teaser?: string;    // max 100 characters
  keywords?: string;  // comma-separated; max 30 characters total
  image?: string;     // URL
}
```

**Response `200`:**

```ts
ApiResponse<null>
```

---

### GET `/api/posts/get/single`

Returns the full post, including the markdown `body`. Used by the `atom-nextjs` SDK to render individual post pages.

**Auth required:** `Authorization: Bearer <project_key>` header

**Query params:**

| Param | Required | Description |
|---|---|---|
| `post_id` | Yes | The `id` of the post |

**Response `200`:**

```ts
ApiResponse<Post>
```

The `post_id` here is the `id` field on the `Post` type (a UUID string), not a MongoDB `_id`.
