# FAQ

## Why do I get "transaction" errors when connecting to MongoDB?

**What you see:** An error like `Transaction numbers are only allowed on a replica set member or mongos` when signing up or creating projects.

**Why it happens:** Several routes (`POST /api/auth/signup`, `POST /api/projects/create`, `DELETE /api/projects/delete`, and `DELETE /api/auth/delete`) use Mongoose transactions via `mongoose.startSession()` and `withTransaction()`. MongoDB transactions require a replica set — a plain standalone `mongod` instance does not support them.

**How to fix it:** Use a MongoDB Atlas cluster instead of a local standalone instance. Atlas free-tier clusters are replica sets by default. If you need a local replica set, you can initialise one with `rs.initiate()` in the `mongosh` shell, or use [mongodb-memory-server](https://github.com/typegoose/mongodb-memory-server) with replica set mode enabled.

---

## Why doesn't dark mode work on my blog?

**What you see:** Post pages and cards don't respond to your site's `dark:` Tailwind classes or `prefers-color-scheme: dark`.

**Why it happens:** The `atom-nextjs` SDK components use Tailwind utility classes that are light-mode only. The prose, border, and background colours used by `Atom`, `AtomPage`, and `AtomPostCard` don't have dark-mode variants. This is a known limitation of the current SDK version.

**Workaround:** There is no supported workaround right now. If dark mode is a requirement, you can replace the SDK components with your own implementations using `getPost` and `getProject` directly:

```ts
import { getPost, getProject } from "atom-nextjs";

// Use the raw data to build your own dark-mode-compatible UI
const project = await getProject(process.env.ATOM_PROJECT_KEY!);
const post = await getPost(process.env.ATOM_PROJECT_KEY!, params.id);
```

---

## How do I run the SDK against a local dev server instead of the live API?

**What you see:** Your local changes to the Atom CMS aren't reflected in your consumer app, even though both are running locally.

**Why it happens:** The `atom-nextjs` package has a hardcoded `baseAPIRoute` in `packages/atom-nextjs/src/lib/constants.ts` that points to `https://cmsatom.netlify.app/api`. The SDK always calls the live production API, regardless of what environment your consumer app is running in.

**How to fix it:** To test against a local server, you need to build and link the package locally:

1. In `packages/atom-nextjs/src/lib/constants.ts`, temporarily change `baseAPIRoute` to `http://localhost:3000/api`.
2. Build the package: `cd packages/atom-nextjs && npm run build`.
3. Link it: `npm link` inside `packages/atom-nextjs`, then `npm link atom-nextjs` in your consumer app.

Remember to revert the `baseAPIRoute` change before publishing a new package version.

---

## What are the plan limits?

All new accounts are on the **Single** plan. Here's what each plan allows:

| Plan | Max projects | Max posts per project | Max body length |
|---|---|---|---|
| Single | 2 | 100 | 10,000 characters |
| Startup | 3 | 1,000 | 100,000 characters |
| Business | 5 | 2,500 | 500,000 characters |

**Startup and Business plans are currently disabled.** The billing UI does not allow upgrading to these plans. All accounts are effectively on the Single plan.

If you hit a limit, the API returns `success: false` with a message like `"Cannot create more than 2 projects."`.

---

## How do I rotate a project key?

**Short answer:** You can't, through the current API.

**Why it happens:** Project keys are generated once at project creation using `crypto.randomBytes(32)` and stored in the `projects` collection. There is no `PATCH /api/projects/update` endpoint or dashboard UI for regenerating a key.

**Workaround:** If you need to invalidate a key, delete the project and create a new one with the same title. You'll get a fresh key, but you'll also lose all the posts in the old project — so make sure to save your post content first.

---

## Why does the CMS say "cmsatom.netlify.app" but the live site is "atomcms.vercel.app"?

**What you see:** The SDK's hardcoded API base URL (`packages/atom-nextjs/src/lib/constants.ts`) points to `https://cmsatom.netlify.app/api`, but the CMS dashboard is hosted at `https://atomcms.vercel.app`.

**Why it happens:** The project has been migrated between hosting providers but the SDK's `baseAPIRoute` hasn't been updated to match. The comment in the constants file even notes it should be changed to `https://www.atomcms.dev/api`. Both URLs currently serve the same application, so the SDK still works, but this inconsistency is a known issue.

**Impact:** No action needed for SDK consumers — the `cmsatom.netlify.app` endpoint responds correctly. If you're building or modifying the SDK itself, update `baseAPIRoute` to the correct canonical URL before publishing.

---

## Why does the dashboard app call its own API over HTTP instead of calling route handlers directly?

**What you see:** Server components and route handlers in the CMS dashboard (e.g. `fetchUser`) use Axios to call `http://localhost:3000/api/*` rather than importing and calling the handler functions directly.

**Why it happens:** The CMS dashboard uses the same REST API that SDK consumers use. Keeping the client helpers as HTTP calls means the API surface is always exercised the same way regardless of whether the caller is the dashboard or an external app. The `baseAPIRoute` in `lib/contants.tsx` is set to `http://localhost:3000/api` in development and `https://cmsatom.netlify.app/api` in production via the `NEXT_PUBLIC_ENV` environment variable.
