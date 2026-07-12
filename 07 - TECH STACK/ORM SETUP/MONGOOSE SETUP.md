# MONGOOSE SETUP [ ODM / MONGODB SCHEMAS ]
------------------------------------------------------------------------

Mongoose is an ODM (Object Document Mapper) for MongoDB. It adds
schemas, validation, and typed models on top of the flexible document
model. Pairs naturally with MongoDB Atlas in a Next.js App Router app.

------------------------------------------------------------------------

## STEP 1 : Install

```bash
pnpm add mongoose
```

------------------------------------------------------------------------

## STEP 2 : Configure Environment

```bash
# .env.local
MONGODB_URI="mongodb+srv://user:password@cluster0.abcde.mongodb.net/dbname?retryWrites=true&w=majority"
```

------------------------------------------------------------------------

## STEP 3 : Connect Singleton For Next.js

Serverless functions and dev hot reload re-run modules constantly. A
naive `mongoose.connect()` opens a new connection each time. Cache the
connection and its in-flight promise on `globalThis`.

```ts
// lib/mongoose.ts
import mongoose from "mongoose";

const uri = process.env.MONGODB_URI!;

const cached = (globalThis as any).mongoose ?? {
  conn: null,
  promise: null,
};
(globalThis as any).mongoose = cached;

export async function dbConnect() {
  if (cached.conn) return cached.conn;
  if (!cached.promise) {
    cached.promise = mongoose.connect(uri, { bufferCommands: false });
  }
  cached.conn = await cached.promise;
  return cached.conn;
}
```

```text
+-------------------------------------------------+
| request -> dbConnect()                          |
|   conn cached?  --yes-->  reuse connection      |
|        |no                                      |
|   promise cached? --no--> mongoose.connect()    |
|        |                  (store promise)       |
|   await promise -> cache conn -> return         |
+-------------------------------------------------+
```

------------------------------------------------------------------------

## STEP 4 : Define Schemas And Models

Guard model creation so hot reload does not redefine an existing model
(`OverwriteModelError`).

```ts
// models/User.ts
import mongoose, { Schema, model, models } from "mongoose";

const UserSchema = new Schema(
  {
    email: { type: String, required: true, unique: true },
    name: { type: String },
    posts: [{ type: Schema.Types.ObjectId, ref: "Post" }],
  },
  { timestamps: true }
);

export const User = models.User ?? model("User", UserSchema);
```

------------------------------------------------------------------------

## STEP 5 : Querying

```ts
// app/api/users/route.ts
import { dbConnect } from "@/lib/mongoose";
import { User } from "@/models/User";

export async function GET() {
  await dbConnect();
  const users = await User.find().limit(10).lean();
  return Response.json(users);
}

export async function POST(req: Request) {
  await dbConnect();
  const { email, name } = await req.json();
  const user = await User.create({ email, name });
  return Response.json(user, { status: 201 });
}
```

Call `dbConnect()` at the top of every route/server action before you
touch a model. Use `.lean()` for read-only queries to skip hydrating
full Mongoose documents (faster, plain objects).

------------------------------------------------------------------------

## WHEN TO USE MONGOOSE (WITH MONGODB)

- You use MongoDB and want schema validation and structure on top.
- You want typed models, hooks (pre/post save), and populate for refs.
- Your documents follow a consistent shape you want enforced in code.
- You prefer an ODM over hand-writing driver queries.

Skip Mongoose and use the raw `mongodb` driver when you want maximum
control, minimal overhead, or truly free-form documents. Use a SQL ORM
(Prisma/Drizzle) if your data is relational rather than document-based.

------------------------------------------------------------------------

## BEST PRACTICES

```text
✓ Cache the connection + promise on globalThis
✓ Set bufferCommands: false so calls fail fast when disconnected
✓ Guard models with models.X ?? model(...) to avoid overwrite errors
✓ Call dbConnect() at the start of each route or server action
✓ Use .lean() for read-only queries to return plain objects
✓ Add timestamps: true for automatic createdAt / updatedAt
✓ Define indexes in the schema for fields you query often
✓ Keep MONGODB_URI in env vars, never in source
```
