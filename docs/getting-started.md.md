# Getting Started

Set up a local Atom instance, create an account, and publish your first blog post.

## Prerequisites

Before you begin, make sure you have these installed and available:

- **Node.js 18+** (with npm or bun)
- **Git**
- **A MongoDB instance** (local or hosted, e.g. [MongoDB Atlas](https://www.mongodb.com/atlas))
- **An Upstash Redis database** (free tier at [upstash.com](https://upstash.com))

## 1. Clone the repository

```bash
git clone https://github.com/incogiscool/atom.git
cd atom
```

## 2. Install dependencies

```bash
npm install
```

Or, if you prefer bun:

```bash
bun install
```

## 3. Configure environment variables

Create a `.env` file in the project root with these five variables:

```env
MONGO_DB_URI=mongodb+srv://your-user:your-password@cluster.mongodb.net/atom
UPSTASH_REDIS_REST_URL=https://your-redis-instance.upstash.io
UPSTASH_REDIS_REST_TOKEN=your-upstash-token
HASH_SALT=any-random-string-for-password-hashing
NEXT_PUBLIC_ENV=dev
```

Here's what each one does:

- **`MONGO_DB_URI`** connects to your MongoDB database. Atom stores user credentials, user documents, projects, and sessions across four collections.
- **`UPSTASH_REDIS_REST_URL`** and **`UPSTASH_REDIS_REST_TOKEN`** power the API rate limiter. Every `/api` request goes through a sliding window limiter (30 requests per minute per IP).
- **`HASH_SALT`** is appended to passwords before hashing with Argon2. Pick a long, random string and don't change it after users have signed up.
- **`NEXT_PUBLIC_ENV`** controls the API base URL. Set it to `dev` for local development (uses `http://localhost:3000/api`). Set it to `prod` for production (uses `https://cmsatom.netlify.app/api`).

If any of the MongoDB or Redis variables are missing, the server will throw on startup with a clear error message.

## 4. Start the dev server

```bash
npm run dev
```

The app starts at [http://localhost:3000](http://localhost:3000). You should see the Atom marketing page.

## 5. Create an account

Navigate to [http://localhost:3000/signup](http://localhost:3000/signup) and fill in the sign-up form:

- **First name** and **Last name** (max 30 characters each)
- **Email** (must be a valid email format)
- **Password** (minimum 8 characters)

On success, you'll be redirected to the dashboard at `/app`. Atom creates two database records in a single transaction: a `credentials` document (storing your Argon2-hashed password) and a `documents` document (your user profile). Both share the same UUID.

New accounts are assigned the **Single** plan (free), which includes up to 2 projects and 100 total posts.

## 6. Create your first project

From the dashboard at `/app`, create a new project by giving it a title (up to 50 characters). Atom generates a unique `project_key` for the project, which external apps use as a Bearer token to fetch your blog content through the API.

After creating the project, you'll see it listed on the dashboard. Click into it to open the project view where you can manage posts.

## 7. Create your first post

Inside your project, create a new post. A post requires:

- **Title** (the headline of your blog post)
- **Author** (the display name for the author)
- **Body** (MDX content, up to 10,000 characters on the Single plan)
- **Teaser** (a short summary for post listings)

Optional fields:

- **Image** (a URL for the post's cover image)
- **Keywords** (comma-separated tags)

Once saved, your post is immediately available through the public API. Any external Next.js app using the `atom-nextjs` SDK with your project key can fetch and render it.

## Verify it's working

To confirm everything is connected, you can hit the API directly. Grab your `project_key` from the project settings in the dashboard, then run:

```bash
curl http://localhost:3000/api/projects/get/single/client \
  -H "Authorization: Bearer YOUR_PROJECT_KEY"
```

You should get back a JSON response containing your project and its posts:

```json
{
  "success": true,
  "message": "Found project.",
  "response": {
    "title": "My First Project",
    "posts": [
      {
        "id": "...",
        "title": "My First Post",
        "author": "Your Name",
        "teaser": "A short summary",
        "image": null,
        "createdAt": "...",
        "updatedAt": "..."
      }
    ]
  }
}
```

Notice the response strips out sensitive fields like `project_key`, `creator_uid`, `body`, and `keywords`. The full post body is only returned when you fetch a single post by ID.

## Next steps

- Read the [Architecture](architecture.md.md) page to understand how the dashboard, API, and SDK fit together.
- Check the [API Reference](api-reference.md.md) for the full list of endpoints.
- Follow the [SDK Guide](sdk-guide.md.md) to render your blog content in an external Next.js app.
