# API Reference

Complete reference for Atom's REST API. All endpoints live under `/api` and return JSON in a consistent response format.

## Base URL

In development: `http://localhost:3000/api`
In production: `https://cmsatom.netlify.app/api`

The base URL is determined by the `NEXT_PUBLIC_ENV` environment variable (`dev` or `prod`).

## Response format

Every endpoint returns the same `ApiResponse<T>` shape:

```json
{
  "success": true,
  "message": "Description of what happened.",
  "response": { }
}
```

When `success` is `false`, `response` is always `null` and `message` contains the error description:

```json
{
  "success": false,
  "message": "Invalid session. Please sign in.",
  "response": null
}
```

## Authentication

Atom uses two authentication methods depending on the endpoint:

- **Session cookie**: For dashboard operations (managing projects, posts, user settings). Lucia Auth sets an `auth_session` cookie on sign-in/sign-up.
- **Bearer token**: For public API endpoints consumed by external apps. Pass the `project_key` in the Authorization header: `Authorization: Bearer <project_key>`.

## Rate limiting

All `/api` routes are rate-limited to **30 requests per minute per IP address** using an Upstash Redis sliding window. Exceeding the limit returns:

```json
{
  "success": false,
  "message": "Too many requests.",
  "response": null
}
```

---

## Auth endpoints

### POST /api/auth/signup

Creates a new user account and sets a session cookie.

**Auth:** None

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | `string` | Yes | Must be a valid email format. Must be unique. |
| `password` | `string` | Yes | Minimum 8 characters. |
| `first_name` | `string` | Yes | Max 30 characters. |
| `last_name` | `string` | Yes | Max 30 characters. |

**Example request:**

```bash
curl -X POST http://localhost:3000/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "securepass123",
    "first_name": "Jane",
    "last_name": "Doe"
  }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Created account successfuly.",
  "response": {
    "_id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "first_name": "Jane",
    "last_name": "Doe",
    "plan": "single",
    "projects": [],
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

The response also sets an `auth_session` cookie. New accounts are assigned the `single` (free) plan.

**Error responses:**

| Message | Cause |
|---|---|
| `"Email not valid."` | Email failed format validation |
| `"Please enter a password."` | Password field empty |
| `"Password must be at least 8 characters long."` | Password too short |
| `"Invalid first name."` | Missing or exceeds 30 characters |
| `"Invalid last name."` | Missing or exceeds 30 characters |
| `"Email already in use."` | Duplicate email (MongoDB error code 11000) |

---

### POST /api/auth/signin

Authenticates an existing user and sets a session cookie.

**Auth:** None

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `email` | `string` | Yes | Registered email address |
| `password` | `string` | Yes | Account password |

**Example request:**

```bash
curl -X POST http://localhost:3000/api/auth/signin \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "password": "securepass123"
  }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Signed in successfuly.",
  "response": {
    "_id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "first_name": "Jane",
    "last_name": "Doe",
    "plan": "single",
    "projects": [],
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

**Error responses:**

| Message | Cause |
|---|---|
| `"Account does not exist."` | No credentials found for that email |
| `"Password is incorrect."` | Argon2 hash comparison failed |

---

### POST /api/auth/signout

Invalidates the current session and clears the session cookie.

**Auth:** Session cookie

**Request body:** None (empty POST)

**Example request:**

```bash
curl -X POST http://localhost:3000/api/auth/signout \
  -b "auth_session=YOUR_SESSION_ID"
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly signed out user.",
  "response": null
}
```

**Error responses:**

| Message | Cause |
|---|---|
| `"Not authorized."` | No valid session cookie |

---

### DELETE /api/auth/delete

Permanently deletes the user account, all their projects, all posts, and all sessions. Requires password confirmation.

**Auth:** Session cookie

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `password` | `string` | Yes | Current password for confirmation |

**Example request:**

```bash
curl -X DELETE http://localhost:3000/api/auth/delete \
  -b "auth_session=YOUR_SESSION_ID" \
  -H "Content-Type: application/json" \
  -d '{ "password": "securepass123" }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly deleted user.",
  "response": null
}
```

This operation runs in a MongoDB transaction. It deletes records from all four collections: `credentials`, `documents`, `projects`, and `sessions`.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No valid session |
| `"Could not find user."` | Credential document missing |
| `"Invalid password."` | Password confirmation failed |

---

### GET /api/auth/user/get

Returns the authenticated user's profile document.

**Auth:** Session cookie

**Query parameters:** None

**Example request:**

```bash
curl http://localhost:3000/api/auth/user/get \
  -b "auth_session=YOUR_SESSION_ID"
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly fetched user document.",
  "response": {
    "_id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "first_name": "Jane",
    "last_name": "Doe",
    "plan": "single",
    "projects": [
      {
        "id": "project-uuid",
        "title": "My Blog",
        "createdAt": "2024-01-15T10:30:00.000Z",
        "updatedAt": "2024-01-15T10:30:00.000Z",
        "creator": {
          "uid": "550e8400-e29b-41d4-a716-446655440000",
          "email": "user@example.com"
        }
      }
    ],
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

The `projects` array contains denormalized summaries, not full project documents.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No valid session |
| `"Error fetching user data."` | User document not found in database |

---

### PATCH /api/auth/user/update

Updates the authenticated user's name fields.

**Auth:** Session cookie

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `first_name` | `string` | No | New first name (max 30 characters) |
| `last_name` | `string` | No | New last name (max 30 characters) |

Both fields are optional. Only provided fields are updated.

**Example request:**

```bash
curl -X PATCH http://localhost:3000/api/auth/user/update \
  -b "auth_session=YOUR_SESSION_ID" \
  -H "Content-Type: application/json" \
  -d '{ "first_name": "Janet" }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly updated user.",
  "response": null
}
```

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No valid session |
| `"First name cannot be longer than 30 characters."` | First name too long |
| `"Last name cannot be longer than 30 characters."` | Last name too long |

---

## Project endpoints

### POST /api/projects/create

Creates a new project with a generated `project_key`.

**Auth:** Session cookie

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | `string` | Yes | Project title (max 50 characters, cannot be empty) |

**Example request:**

```bash
curl -X POST http://localhost:3000/api/projects/create \
  -b "auth_session=YOUR_SESSION_ID" \
  -H "Content-Type: application/json" \
  -d '{ "title": "My Engineering Blog" }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly created project.",
  "response": {
    "_id": "project-uuid",
    "title": "My Engineering Blog",
    "posts": [],
    "project_key": "atom-base64encodedstring...",
    "creator_uid": "user-uuid",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

The `project_key` is a 32-byte random value, base64-encoded, prefixed with `atom-`. This key is what external apps use as a Bearer token.

The operation runs in a MongoDB transaction: it creates the project document and pushes a summary into the user's `projects` array.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid title."` | Empty, missing, or exceeds 50 characters |
| `"Invalid session. Please sign in."` | No valid session |
| `"Could not find user."` | User document not found |
| `"Cannot create more than N projects."` | Plan project limit reached (2 for Single) |

---

### DELETE /api/projects/delete

Deletes a project and all its embedded posts.

**Auth:** Session cookie

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `project_id` | `string` | Yes | The project's `_id` |

**Example request:**

```bash
curl -X DELETE "http://localhost:3000/api/projects/delete?project_id=PROJECT_UUID" \
  -b "auth_session=YOUR_SESSION_ID"
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly deleted project.",
  "response": null
}
```

Runs in a transaction: deletes the project document and removes the summary from the user's `projects` array.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No valid session |
| `"Invalid project id."` | Missing parameter or project not found |
| `"Not authorized."` | User doesn't own this project |

---

### GET /api/projects/get/single

Returns the full project document. Supports two auth methods: session cookie (for dashboard use) or Bearer token (for external use).

**Auth:** Session cookie OR Bearer token

**With session cookie:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `project_id` | `string` | Yes | The project's `_id` |

```bash
curl "http://localhost:3000/api/projects/get/single?project_id=PROJECT_UUID" \
  -b "auth_session=YOUR_SESSION_ID"
```

**With Bearer token:**

```bash
curl http://localhost:3000/api/projects/get/single \
  -H "Authorization: Bearer YOUR_PROJECT_KEY"
```

When using a Bearer token, no `project_id` parameter is needed because the key uniquely identifies the project.

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly fetched posts.",
  "response": {
    "_id": "project-uuid",
    "title": "My Engineering Blog",
    "posts": [
      {
        "id": "post-uuid",
        "title": "First Post",
        "author": "Jane Doe",
        "body": "# Hello\n\nThis is MDX content...",
        "image": null,
        "keywords": ["intro"],
        "teaser": "A short summary",
        "creator_uid": "user-uuid",
        "createdAt": "2024-01-15T10:30:00.000Z",
        "updatedAt": "2024-01-15T10:30:00.000Z"
      }
    ],
    "project_key": "atom-base64encodedstring...",
    "creator_uid": "user-uuid",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

This endpoint returns the **full, unsanitized** project, including `project_key`, `creator_uid`, and post bodies. For a sanitized version suitable for public consumption, use `/api/projects/get/single/client`.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No session and no Bearer token |
| `"Invalid project id."` | Missing `project_id` or project not found |
| `"Not authorized."` | Session user doesn't own the project |
| `"Invalid project key."` | Bearer token doesn't match any project |

---

### GET /api/projects/get/single/client

Returns a sanitized project for public consumption. Strips sensitive fields from the project and its posts.

**Auth:** Bearer token

```bash
curl http://localhost:3000/api/projects/get/single/client \
  -H "Authorization: Bearer YOUR_PROJECT_KEY"
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly fetched posts.",
  "response": {
    "title": "My Engineering Blog",
    "id": "project-uuid",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z",
    "posts": [
      {
        "id": "post-uuid",
        "title": "First Post",
        "author": "Jane Doe",
        "teaser": "A short summary",
        "image": null,
        "createdAt": "2024-01-15T10:30:00.000Z",
        "updatedAt": "2024-01-15T10:30:00.000Z"
      }
    ]
  }
}
```

Compared to `/api/projects/get/single`, this endpoint removes:
- `project_key` and `creator_uid` from the project
- `body`, `keywords`, and `creator_uid` from each post

This is the endpoint the SDK's `getProject()` function calls.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid project key."` | Missing or empty Authorization header |
| `"Must be bearer token."` | Authorization header isn't `Bearer <key>` |
| `"Could not find project."` | No project matches the key |

---

## Post endpoints

### POST /api/posts/create

Creates a new post inside a project.

**Auth:** Session cookie

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `project_id` | `string` | Yes | The project to add the post to |

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | `string` | Yes | Post title |
| `author` | `string` | Yes | Author display name |
| `body` | `string` | Yes | MDX content (max length depends on plan) |
| `teaser` | `string` | Yes | Short summary for post listings |
| `image` | `string` | No | Cover image URL |
| `keywords` | `string` | No | Comma-separated tags (stored as array) |

The `keywords` field is a comma-separated string in the request. The server splits it into an array before storing.

**Example request:**

```bash
curl -X POST "http://localhost:3000/api/posts/create?project_id=PROJECT_UUID" \
  -b "auth_session=YOUR_SESSION_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Getting Started with Atom",
    "author": "Jane Doe",
    "body": "# Introduction\n\nWelcome to Atom...",
    "teaser": "Learn how to set up Atom CMS",
    "image": "https://example.com/image.png",
    "keywords": "tutorial,getting-started,cms"
  }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly created post.",
  "response": {
    "id": "post-uuid",
    "title": "Getting Started with Atom",
    "author": "Jane Doe",
    "body": "# Introduction\n\nWelcome to Atom...",
    "teaser": "Learn how to set up Atom CMS",
    "image": "https://example.com/image.png",
    "keywords": ["tutorial", "getting-started", "cms"],
    "creator_uid": "user-uuid",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

**Body length limits by plan:**

| Plan | Max body length |
|---|---|
| Single (free) | 10,000 characters |
| Startup | 100,000 characters |
| Business | 500,000 characters |

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid project id."` | Missing `project_id` parameter |
| `"Invalid session. Please sign in."` | No valid session |
| `"Not authorized."` | Session user doesn't own the project |
| `"Body cannot be more than N characters."` | Body exceeds plan limit |

---

### DELETE /api/posts/delete

Deletes a post from a project.

**Auth:** Session cookie

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `project_id` | `string` | Yes | The project containing the post |
| `post_id` | `string` | Yes | The post to delete |

**Example request:**

```bash
curl -X DELETE "http://localhost:3000/api/posts/delete?project_id=PROJECT_UUID&post_id=POST_UUID" \
  -b "auth_session=YOUR_SESSION_ID"
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly deleted post.",
  "response": null
}
```

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No valid session |
| `"Invalid project id."` | Missing parameter or project not found |
| `"Not authorized."` | Session user doesn't own the project |
| `"Could not find post."` | No post with that ID in the project |

---

### PATCH /api/posts/update

Updates fields on an existing post. Only provided fields are changed.

**Auth:** Session cookie

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `project_id` | `string` | Yes | The project containing the post |
| `post_id` | `string` | Yes | The post to update |

**Request body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `title` | `string` | No | New title (max 50 characters) |
| `author` | `string` | No | New author name (max 30 characters) |
| `body` | `string` | No | New MDX content (max depends on plan) |
| `teaser` | `string` | No | New teaser (max 100 characters) |
| `keywords` | `string` | No | New comma-separated keywords (max 30 characters) |
| `image` | `string` | No | New cover image URL |

All fields are optional. Empty strings and null values are filtered out, so only non-empty fields get updated.

**Example request:**

```bash
curl -X PATCH "http://localhost:3000/api/posts/update?project_id=PROJECT_UUID&post_id=POST_UUID" \
  -b "auth_session=YOUR_SESSION_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Title",
    "body": "# Updated Content\n\nNew body text..."
  }'
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly updated post",
  "response": null
}
```

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid session. Please sign in."` | No valid session |
| `"Invalid project id."` | Missing parameter or project not found |
| `"Not authorized."` | Session user doesn't own the project |
| `"Could not find post."` | No post with that ID |
| `"Title cannot be longer than 50 characters."` | Title too long |
| `"Invalid author."` | Author exceeds 30 characters |
| `"Invalid teaser."` | Teaser exceeds 100 characters |
| `"Invalid keywords."` | Keywords string exceeds 30 characters |
| `"Body cannot be more than N characters."` | Body exceeds plan limit |

---

### GET /api/posts/get/single

Returns a single post with full body content. This is the endpoint the SDK's `getPost()` function calls.

**Auth:** Bearer token

**Query parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `post_id` | `string` | Yes | The post ID to fetch |

**Example request:**

```bash
curl "http://localhost:3000/api/posts/get/single?post_id=POST_UUID" \
  -H "Authorization: Bearer YOUR_PROJECT_KEY"
```

**Success response:**

```json
{
  "success": true,
  "message": "Successfuly fetched post.",
  "response": {
    "id": "post-uuid",
    "title": "Getting Started with Atom",
    "author": "Jane Doe",
    "body": "# Introduction\n\nWelcome to Atom...\n\nThis post was created using [Atom](https://atomcms.vercel.app/)",
    "teaser": "Learn how to set up Atom CMS",
    "image": "https://example.com/image.png",
    "keywords": ["tutorial", "getting-started", "cms"],
    "creator_uid": "user-uuid",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

On the Single (free) plan, the server appends a Markdown watermark to the body: `"\n\nThis post was created using [Atom](https://atomcms.vercel.app/)"`. Paid plans do not include this watermark.

**Error responses:**

| Message | Cause |
|---|---|
| `"Invalid project key."` | Missing or empty Authorization header |
| `"Must be bearer token."` | Authorization header isn't `Bearer <key>` |
| `"Could not find project."` | No project matches the key |
| `"Could not find post."` | No post with that ID in the project |
| `"Owner user does not exist."` | Project owner's user document is missing |
