# Open Questions

## Architecture & Design
1. **Why does the app use itself as the SDK consumer?** — The main app has `atom-nextjs` as a dependency and renders its own blog using the SDK (`app/blog/`). Is this intentional dogfooding, or is the blog route meant to be extracted?

2. **Deployment target?** — The codebase references both `https://cmsatom.netlify.app` (in constants and SDK) and `https://atomcms.vercel.app` (in README watermark). Which is the current production URL? The `NEXT_PUBLIC_ENV` variable toggles between dev/prod base URLs.

3. **Billing/payments not implemented** — The billing page (`app/app/settings/billing/page.tsx`) shows "coming soon". The `startup` and `business` plans are `disabled: true`. Is there a plan for payment integration (Stripe, etc.)?

4. **Zustand and React Query installed but unused?** — Both `zustand` and `@tanstack/react-query` are in dependencies but don't appear actively used in the codebase. Were these for planned features?

## Security
5. **Password salt in env variable** — The `HASH_SALT` env var is appended to passwords before Argon2 hashing (`lib/server/encoding/encodePassword.ts`). Argon2 already generates unique salts per hash. Is this extra salt intentional or redundant? The current implementation means all users share the same pre-salt.

6. **No CSRF protection** — API routes using session cookies don't appear to have CSRF token validation. Is this a concern?

7. **Rate limit on all API routes uniformly** — The middleware applies the same 30 req/min limit to all `/api` routes. Should public SDK endpoints have different limits than dashboard endpoints?

## Data Model
8. **Denormalized project data in UserDocument** — Projects are stored both as full documents in the `projects` collection and as summaries in `UserDocument.projects[]`. Consistency between these could drift. Is this by design for read performance?

9. **No pagination** — Posts are embedded as an array within projects. With up to 2,500 posts (business plan), this could get large. Is pagination planned?

## Testing
10. **No tests exist** — There are zero test files in both the main app and the SDK. Is testing planned? What areas would be priority?

## SDK
11. **SDK hardcoded API URL** — The SDK's `baseAPIRoute` is hardcoded to `https://cmsatom.netlify.app/api` in `packages/atom-nextjs/src/lib/constants.ts`. This can't be configured by consumers. Is this intentional?

12. **SDK does not support dark mode** — Per the README: "Currently, Atom does not support websites with dark mode." Is this a planned feature?

13. **SDK types duplicated** — The SDK duplicates `Post`, `ClientPost`, `ClientProject`, and `ApiResponse` types from the main app rather than sharing them. Could these be shared via a common package?
