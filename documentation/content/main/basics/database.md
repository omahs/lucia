---
title: "Database"
description: "Learn how your database works with Lucia"
---

A database is required for storing your users and sessions. Lucia connects to your database via an adapter, which provides a set of basic, standardized querying methods that Lucia can use.

## Database adapters

There are 2 types of adapters provided by Lucia: Regular adapters, and session adapters. As the name implies, session adapters only handles queries to the session table. This useful for when you want to store your sessions in a different database then your users, such as Redis and other memory stores.

We currently provide the following adapters:

- [`better-sqlite3`](/database-adapters/better-sqlite3)
- [libSQL](/database-adapters/libsql)
- [Cloudflare D1](/database-adapters/cloudflare-d1)
- [Mongoose](/database-adapters/mongoose)
- [`mysql2`](/database-adapters/mysql2)
- [`pg`](/database-adapters/pg)
- [`postgres`](/database-adapters/postgres)
- [PlanetScale serverless](/database-adapters/planetscale-serverless)
- [Prisma](/database-adapters/prisma)
- [Redis](/database-adapters/redis)
- [Unstorage](/database-adapters/unstorage)

SDKs such as `@vercel/postgres` and `@neonserverless/database` provide drop-in replacements for existing drivers. You can also use query builders like Drizzle ORM and Kysely since they rely on underlying drivers that we provide adapters for. Refer to these guides:

- [Using `@vercel/postgres`](/guidebook/vercel-postgres)
- [Using Drizzle ORM](/guidebook/drizzle-orm)
- [Using Kysely](/guidebook/kysely)

## Database model

Lucia requires 3 tables to work, which are then connected to Lucia via a database adapter. This is only the basic model and the specifics depend on the adapter.

### User table

| name | type     | primary key | description |
| ---- | -------- | :---------: | ----------- |
| id   | `string` |      ✓      | User id     |

```ts
type UserSchema = {
	id: string;
} & Lucia.DatabaseUserAttributes;
```

In addition to the required fields shown below, you can add any additional fields to the table, in which case they should be declared in type `Lucia.DatabaseUserAttributes`.

```ts
declare namespace Lucia {
	// ...
	type DatabaseUserAttributes = {
		// required fields (i.e. id) should not be defined here
		username: string;
	};
}
```

### Session table

| name           | type            | primary key | references | description                                        |
| -------------- | --------------- | :---------: | ---------- | -------------------------------------------------- |
| id             | `string`        |      ✓      |            |                                                    |
| user_id        | `string`        |             | `user(id)` |                                                    |
| active_expires | `number` (int8) |             |            | The expiration time (unix) of the session (active) |
| idle_expires   | `number` (int8) |             |            | The expiration time (unix) for the idle period     |

```ts
type SessionSchema = {
	id: string;
	active_expires: number;
	idle_expires: number;
	user_id: string;
} & Lucia.DatabaseSessionAttributes;
```

In addition to the required fields shown below, you can add any additional fields to the table, in which case they should be declared in type `Lucia.DatabaseSessionAttributes`.

```ts
declare namespace Lucia {
	// ...
	type DatabaseSessionAttributes = {
		// required fields (i.e. id) should not be defined here
		username: string;
	};
}
```

### Key table

| name            | type             | primary key | references | description                                              |
| --------------- | ---------------- | :---------: | ---------- | -------------------------------------------------------- |
| id              | `string`         |      ✓      |            | Key id in the form of: `${providerId}:${providerUserId}` |
| user_id         | `string`         |             | `user(id)` |                                                          |
| hashed_password | `string \| null` |             |            | Hashed password of the key                               |

```ts
type KeySchema = {
	id: string;
	user_id: string;
	hashed_password: string | null;
};
```
