# Authentication

This page explains how Atom handles authentication and authorization: how sessions work, how passwords are stored, how routes are protected, and how rate limiting fits in.

## Overview

Atom uses [Lucia Auth](https://lucia-auth.com/) v3 for session-based authentication. There are no JWTs and no third-party OAuth providers. Users sign up with an email and password, and the server manages sessions through HTTP cookies.

The authentication system involves four pieces:

1. **Lucia** manages session creation, validation, and invalidation
2. **Argon2** hashes passwords before storing them
3. **ProtectedRoute** is a server component that gates access to dashboard pages
4. **Rate limiting** (Upstash Redis) protects API routes from abuse

## Password hashing

Atom never stores plaintext passwords. When a user signs up or signs in, the password goes through this process:

```typescript
// lib/server/encoding/encodePassword.ts
import argon2 from "argon2";

export const encodePassword = async (password: string) => {
  const salt = process.env.HASH_SALT;
  const saltedString = password + salt;
  const hash = await argon2.hash(saltedString);
  return hash;
};
```

The password is concatenated with a salt from the `HASH_SALT` environment variable, then hashed with Argon2. Argon2 is a memory-hard hashing algorithm, which means it's expensive to brute-force even with specialized hardware.

Verification works the same way in reverse:

```typescript
// lib/server/encoding/isPasswordValid.ts
import argon2 from "argon2";

export const isPasswordValid = async (hash: string, password: string) => {
  return await argon2.verify(hash, password + process.env.HASH_SALT);
};
```

The salt is appended to the password before comparing against the stored hash. If `HASH_SALT` changes after users have signed up, all existing passwords will fail verification. Treat it as permanent once set.

## Session management

Lucia Auth handles the session lifecycle. Here's how it's initialized (`lib/server/lucia/init.ts`):

```typescript
import { Lucia } from "lucia";
import { MongodbAdapter } from "@lucia-auth/adapter-mongodb";
import mongoose from "mongoose";

export const adapter = new MongodbAdapter(
  mongoose.connection.collection("sessions"),
  mongoose.connection.collection("credentials")
);

export const lucia = new Lucia(adapter, {
  sessionCookie: {
    expires: false,
    attributes: {
      secure: process.env.NODE_ENV === "production",
    },
  },
  getUserAttributes: (attributes) => {
    return {
      email: attributes.email,
    };
  },
});
```

Key configuration choices:

- **MongoDB adapter** connects Lucia to the `sessions` and `credentials` collections. Lucia reads user data from `credentials` and stores sessions in `sessions`.
- **`expires: false`** means session cookies are persistent (not cleared when the browser closes). Sessions still expire server-side based on Lucia's default expiration.
- **`secure: true` in production** ensures session cookies are only sent over HTTPS.
- **`getUserAttributes`** extracts the `email` field from the credentials document, making it available on the `user` object after session validation.

### Signup flow

When a user signs up (`app/api/auth/signup/route.ts`):

1. The server validates the email format, password length (min 8 chars), and name fields (max 30 chars each).
2. The password is hashed with Argon2 + salt.
3. A UUID is generated for the new user.
4. In a single MongoDB transaction, the server creates both a `credentials` document and a `documents` document with the same `_id`.
5. Lucia creates a new session and the server sets the session cookie on the response.

### Signin flow

When a user signs in (`app/api/auth/signin/route.ts`):

1. The server looks up the `credentials` document by email.
2. It verifies the password against the stored hash using `isPasswordValid()`.
3. It fetches the matching `documents` record to include in the response.
4. Lucia creates a new session and the server sets the session cookie.

### Signout flow

When a user signs out (`app/api/auth/signout/route.ts`):

1. The server validates the current session.
2. Lucia invalidates the session (deletes it from the `sessions` collection).
3. The server sets a blank session cookie, which clears the old one.

## Session validation

Every authenticated request goes through `validateRequest()` (`lib/server/lucia/functions/validate-request.ts`):

```typescript
import { cookies } from "next/headers";
import { cache } from "react";
import type { Session, User } from "lucia";
import { lucia } from "../init";
import { connectToDatabase } from "../../mongo/init";

export const validateRequest = cache(
  async (): Promise<
    { user: User; session: Session } | { user: null; session: null }
  > => {
    await connectToDatabase();

    const sessionId = cookies().get(lucia.sessionCookieName)?.value ?? null;
    if (!sessionId) {
      return { user: null, session: null };
    }

    const result = await lucia.validateSession(sessionId);

    try {
      if (result.session && result.session.fresh) {
        const sessionCookie = lucia.createSessionCookie(result.session.id);
        cookies().set(
          sessionCookie.name,
          sessionCookie.value,
          sessionCookie.attributes
        );
      }

      if (!result.session) {
        const sessionCookie = lucia.createBlankSessionCookie();
        cookies().set(
          sessionCookie.name,
          sessionCookie.value,
          sessionCookie.attributes
        );
      }
    } catch (err: any) {
      console.error(err);
    }
    return result;
  }
);
```

This function is wrapped in React's `cache()`, which deduplicates calls within a single server-side request. If multiple components or API handlers call `validateRequest()` during the same request, the session is only validated once.

The flow:

1. Read the session cookie from the request headers.
2. If there's no cookie, return `{ user: null, session: null }`.
3. Call `lucia.validateSession()` to check the session against the `sessions` collection.
4. If the session is "fresh" (recently created or renewed), refresh the session cookie to extend its lifetime.
5. If the session is invalid, set a blank cookie to clear it.
6. Return the `user` and `session` objects, or nulls if authentication failed.

## Protected routes

Dashboard pages under `/app/*` are protected by a server component wrapper.

The layout at `app/app/layout.tsx` wraps all child pages:

```typescript
import ProtectedRoute from "@/components/containers/ProtectedRoute";
import { ReactNode } from "react";

export default function AppLayout({ children }: { children: ReactNode }) {
  return <ProtectedRoute>{children}</ProtectedRoute>;
}
```

And `ProtectedRoute` itself (`components/containers/ProtectedRoute.tsx`):

```typescript
import { validateRequest } from "@/lib/server/lucia/functions/validate-request";
import { connectToDatabase } from "@/lib/server/mongo/init";
import { redirect } from "next/navigation";

export default async function ProtectedRoute({
  children,
}: {
  children: React.ReactNode;
}) {
  await connectToDatabase();
  const { user } = await validateRequest();

  if (!user) {
    return redirect("/signin");
  }
  return <>{children}</>;
}
```

This is a server component, so the authentication check happens on the server before any HTML is sent to the client. If the session is invalid, the user is redirected to `/signin` with a server-side redirect (HTTP 307).

## Authorization in API routes

API routes that modify data (creating projects, editing posts, deleting accounts) follow a consistent authorization pattern:

1. Call `validateRequest()` to get the current user from the session cookie.
2. If there's no user, throw `"Invalid session. Please sign in."`.
3. For resource-specific operations, verify ownership by comparing `user.id` to the resource's `creator_uid`.

Here's the pattern as it appears in the project deletion handler:

```typescript
const { user } = await validateRequest();
if (!user) throw new Error("Invalid session. Please sign in.");

const project = await ProjectsRef.findOne({ _id: project_id });
if (!project) throw new Error("Invalid project id.");

const isAuth = project.creator_uid === user.id;
if (!isAuth) throw new Error("Not authorized.");
```

This three-step check (valid session, resource exists, user owns resource) repeats across post creation, post updates, post deletion, and project deletion.

Account deletion adds an extra layer: the user must provide their current password in the request body, which is verified against the stored hash before proceeding.

## Public API authentication

The public API endpoints used by the SDK authenticate differently. Instead of session cookies, they use the project's `project_key` as a Bearer token:

```
Authorization: Bearer atom-base64encodedstring...
```

The server extracts the token, looks up the project by `project_key`, and returns the data. There's no user session involved. This means anyone with the project key can read all posts in the project.

Two endpoints use Bearer auth:
- `GET /api/projects/get/single/client` (sanitized project listing)
- `GET /api/posts/get/single` (full post content)

One endpoint supports both methods:
- `GET /api/projects/get/single` checks for an Authorization header first (Bearer token), and falls back to session cookie + `project_id` query parameter.

## Rate limiting

All `/api` routes pass through the middleware at `middleware.ts` before reaching route handlers. The middleware applies rate limiting using Upstash Redis:

```typescript
import { ratelimit } from "./lib/server/redis/init";

export default async function middleware(request: NextRequest) {
  const ip = request.ip ?? "127.0.0.1";

  if (requestPath.startsWith("/api")) {
    try {
      const { success } = await ratelimit.limit(ip);
      if (!success) throw new Error("Too many requests.");
    } catch (err: any) {
      return NextResponse.json<ApiResponse>({
        response: null,
        success: false,
        message: err.message || err,
      });
    }
  }
}
```

The rate limiter is configured in `lib/server/redis/init.ts`:

```typescript
export const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(30, "1 m"),
});
```

This creates a sliding window of 30 requests per minute per IP address. The sliding window algorithm is smoother than fixed windows because it doesn't have a burst problem at window boundaries.

Rate limiting applies to all API routes equally, including both authenticated dashboard requests and public SDK requests. The middleware matcher is configured to only run on `/api/:path*`, so page navigation and static assets are not rate-limited.

If an IP exceeds the limit, the middleware short-circuits the request and returns a JSON error response without forwarding to the route handler. The limit resets as older requests age out of the sliding window.

## Common misconceptions

**"The session cookie contains user data."** It doesn't. The cookie only contains a session ID. User data is looked up from the database on every request via `validateRequest()`.

**"The project key is a JWT or signed token."** It's not. It's a random string used as a simple database lookup key. There's no cryptographic verification of the token itself; security relies on the key being hard to guess (32 random bytes).

**"Rate limiting protects against credential stuffing."** Partially. The 30 requests/minute limit per IP slows down brute-force attacks, but a determined attacker using multiple IPs could work around it. There's no per-account rate limiting or account lockout mechanism.
