# Page Proposal

## Approved Documentation Structure

1. **docs/getting-started.md** — Guide — Walk a new developer through cloning the repo, setting up environment variables (MongoDB, Redis, Hash Salt), running the dev server, and creating their first project/post.

2. **docs/architecture.md** — Explanation — How the system is organized: Next.js App Router structure, the two-part architecture (dashboard app + SDK package), data flow from dashboard to public API to consumer apps, and key design decisions.

3. **docs/api-reference.md** — Reference — Complete REST API documentation covering all 14 endpoints (auth, posts, projects) with request/response formats, authentication requirements, and error handling.

4. **docs/sdk-guide.md** — Guide — How to integrate the `atom-nextjs` SDK into a consumer Next.js app: installation, setting up blog pages, configuring TailwindCSS, loading states, metadata generation, sitemaps, and caching.

5. **docs/data-model.md** — Reference — MongoDB collections, Mongoose schemas, TypeScript types, entity relationships, and plan limits.

6. **docs/authentication.md** — Explanation — How Lucia Auth works in this project: session management, password hashing with Argon2, protected routes, authorization patterns, and rate limiting.
