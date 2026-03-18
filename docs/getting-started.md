# Getting Started

This guide walks you through running the Atom CMS application locally. By the end you'll have a working dev server, a connected database, and a fresh account you can log into.

## Prerequisites

- Node.js 18 or later
- A [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) cluster (or any MongoDB instance that supports transactions  -  see the note below about replica sets)
- An [Upstash](https://upstash.com) account for Redis

> **MongoDB replica set required.** Several routes use Mongoose transactions (`mongoose.startSession()` + `withTransaction`). Transactions require a replica set. MongoDB Atlas free-tier clusters are replica sets by default. A plain standalone `mongod` will not work.

## 1. Clone the repository

```bash
git clone https://github.com/incogiscool/atom.git
cd atom
```

## 2. Install dependencies

```bash
npm install
```

## 3. Set environment variables

Create a `.env.local` file in the project root. The application needs five variables:

```bash
# MongoDB connection string from Atlas (or your replica set)
MONGO_DB_URI=mongodb+srv://<user>:<password>@cluster0.example.mongodb.net/atom

# A random string appended to every password before hashing with Argon2.
# Pick anything  -  but don't change it after you've created accounts.
HASH_SALT=some-long-random-string

# Set to "prod" to point the CMS dashboard at the production API.
# Leave it unset (or set to anything else) to use http://localhost:3000/api.
NEXT_PUBLIC_ENV=dev

# Upstash Redis credentials  -  find these in your Upstash console.
UPSTASH_REDIS_REST_URL=https://<your-instance>.upstash.io
UPSTASH_REDIS_REST_TOKEN=<your-token>
```

**Where to find Upstash credentials:** Log in to [console.upstash.com](https://console.upstash.com), create a Redis database, then copy the REST URL and REST Token from the database detail page.

**Where to find the MongoDB URI:** In Atlas, click *Connect* on your cluster, choose *Drivers*, and copy the connection string. Replace `<password>` with your database user's password.

## 4. Run the dev server

```bash
npm run dev
```

The app starts on [http://localhost:3000](http://localhost:3000). The first request to any API route will establish the Mongoose connection  -  you'll see `connected to database` in the terminal output.

## 5. Sign up for an account

Open [http://localhost:3000/signup](http://localhost:3000/signup) in your browser. Fill in your name, email, and a password of at least eight characters, then submit. On success, you'll be redirected to the dashboard at `/app`.

You can verify everything is wired up correctly by opening your MongoDB Atlas cluster and confirming that three collections have appeared: `credentials`, `documents`, and `session`.

## Next steps

- Read the [Architecture](./architecture.md) overview to understand how the CMS dashboard and the `atom-nextjs` SDK fit together.
- If you're only interested in adding a blog to an existing Next.js site, skip to the [Quickstart](./quickstart.md).
