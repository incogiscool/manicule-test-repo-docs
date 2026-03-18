# Authentication

Atom uses [Lucia v3](https://lucia-auth.com) for session management. Lucia handles the session lifecycle — creating, validating, refreshing, and invalidating sessions — while the application code owns credential verification (password hashing) and route protection.

## How sessions work

When a user signs in, two things happen server-side:

1. The password is verified against the stored Argon2 hash.
2. `lucia.createSession()` is called, which writes a new record to the `session` MongoDB collection and returns a signed session ID.

That session ID is set as an HTTP-only cookie using `lucia.createSessionCookie()`:

```ts
const session = await lucia.createSession(userDocument._id, {});
const sessionCookie = lucia.createSessionCookie(session.id);
cookies().set(
  sessionCookie.name,
  sessionCookie.value,
  sessionCookie.attributes
);
```

The cookie is configured as non-expiring (`expires: false`) so it persists until the user explicitly signs out or the session is invalidated. In production (`NODE_ENV === "production"`) the cookie's `secure` attribute is set to `true`, restricting it to HTTPS.

## Validating a session on the server

Any server component or route handler that needs to know who's logged in calls `validateRequest()`, defined in `lib/server/lucia/functions/validate-request.ts`:

```ts
export const validateRequest = cache(
  async (): Promise<
    { user: User; session: Session } | { user: null; session: null }
  > => {
    const sessionId = cookies().get(lucia.sessionCookieName)?.value ?? null;
    if (!sessionId) return { user: null, session: null };

    const result = await lucia.validateSession(sessionId);

    // Refresh a "fresh" session cookie, or clear an expired one
    if (result.session?.fresh) {
      cookies().set(/* fresh session cookie */);
    }
    if (!result.session) {
      cookies().set(/* blank cookie */);
    }

    return result;
  }
);
```

The function is wrapped in React's `cache()`. This means within a single server render — even if ten different components call `validateRequest()` — the session is only looked up from MongoDB once. The result is shared across the render.

If the session is still valid but Lucia has marked it as "fresh" (about to be refreshed), the function silently rolls over to a new cookie. If the session has expired or is missing, it writes a blank cookie to clear the stale value in the browser.

## Protecting dashboard routes

All `/app/*` routes go through `ProtectedRoute`, a server component that gates the entire dashboard:

```ts
// components/containers/ProtectedRoute.tsx
export default async function ProtectedRoute({ children }) {
  await connectToDatabase();
  const { user } = await validateRequest();

  if (!user) {
    return redirect("/signin");
  }
  return <>{children}</>;
}
```

It's applied in `app/app/layout.tsx`, which wraps every page under `/app/*`. The redirect happens before any page content is rendered, so unauthenticated requests never reach the dashboard components.

## Sign-out

Signing out calls `lucia.invalidateSession()`, which deletes the session record from MongoDB, and then replaces the cookie with a blank one:

```ts
await lucia.invalidateSession(session.id);
const sessionCookie = lucia.createBlankSessionCookie();
cookies().set(sessionCookie.name, sessionCookie.value, sessionCookie.attributes);
```

## Password hashing

Passwords are hashed with [Argon2](https://github.com/ranisalt/node-argon2) before storage. The application appends an environment-provided salt to the password before hashing:

```ts
// lib/server/encoding/encodePassword.ts
export const encodePassword = async (password: string) => {
  const saltedString = password + process.env.HASH_SALT;
  return await argon2.hash(saltedString);
};
```

Verification reverses the same concatenation:

```ts
// lib/server/encoding/isPasswordValid.ts
export const isPasswordValid = async (hash: string, password: string) => {
  return await argon2.verify(hash, password + process.env.HASH_SALT);
};
```

The `HASH_SALT` value must remain constant for the lifetime of the application — changing it will invalidate all existing passwords. It does not need to be unique per user; Argon2 generates its own random per-hash salt internally. The `HASH_SALT` environment variable is an additional application-level pepper.

## Lucia's MongoDB adapter

Lucia is initialized with a `MongodbAdapter` that points at two Mongoose collections:

```ts
// lib/server/lucia/init.ts
export const adapter = new MongodbAdapter(
  mongoose.connection.collection("sessions"),
  mongoose.connection.collection("credentials")
);
```

Lucia uses `credentials` as its user table (matching the `UserCredentials` Mongoose model) and `sessions` for session storage. The `_id` field in `credentials` is the UUID generated at signup, which becomes the `user_id` in every session record.

## Rate limiting

Every request to `/api/*` passes through the Next.js middleware in `middleware.ts` before the route handler runs:

```ts
// middleware.ts
export default async function middleware(request: NextRequest) {
  const ip = request.ip ?? "127.0.0.1";

  if (requestPath.startsWith("/api")) {
    const { success } = await ratelimit.limit(ip);
    if (!success) throw new Error("Too many requests.");
  }
}

export const config = {
  matcher: ["/api/:path*"],
};
```

The rate limiter uses an Upstash Redis sliding window of 30 requests per minute per IP. It's initialized in `lib/server/redis/init.ts`:

```ts
export const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(30, "1 m"),
});
```

When the limit is exceeded, the middleware returns a JSON `ApiResponse` with `success: false` and `message: "Too many requests."` — the route handler is never called. This applies equally to authenticated and unauthenticated routes.

## Common misconceptions

**"Session cookies expire when the browser closes."** They don't — the cookie is set with `expires: false`, so it persists between browser sessions. Users stay logged in until they explicitly sign out.

**"The project_key and the session cookie are the same thing."** They're not. The session cookie authenticates *dashboard users* (humans managing posts). The `project_key` authenticates *SDK consumers* (Next.js apps fetching published posts). They go through separate code paths and carry different privileges.
