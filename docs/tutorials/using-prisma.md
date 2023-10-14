---
sidebar_position: 10
title: Using Prisma
description: Learn how to create and register Prisma entities and perform transactional updates
---

In this guide, you'll learn how to build an Operon application using [Prisma](https://www.prisma.io/), a popular open-source Typescript ORM.
We'll show you how to build a "Hello, World!" application using Prisma.
The source code for the complete application is [here](https://github.com/dbos-inc/operon-demo-apps/tree/main/hello-prisma).

### Prisma Setup

Before using Prisma to build an application, you have to set up a schema, generate a primsa client, and run migrations to create database tables.
For more information, see [Prisma's getting started guide](https://www.prisma.io/docs/getting-started).

Here's how to complete these steps for our demo app.
Make sure you have a running Postgres database, for example from our [quickstart init script](../getting-started/quickstart).

Create a directory:
```
mkdir hello-prisma
```

Install prisma:
```
npm init -y
npm install prisma typescript ts-node @types/node --save-dev
npx prisma init
```

In the .env file, update the database URL if necessary: `DATABASE_URL="postgresql://postgres:${PGPASSWORD}@localhost:5432/helloprisma?schema=public"`.

In the file `prisma/schema.prisma`, add a model:
```tsx
model OperonHello {
  @@map("operonhello")
  greeting_id Int @id @default(autoincrement())
  greeting String
}
```

Run this command in the database to create tables:
```
npx prisma migrate dev --name init
```

### Configuring Operon to use Prisma

Next, configure Operon to use Prisma by setting the `user_dbclient` property of the [configuration file](../api-reference/configuration) to "prisma".
For example:

```yaml
database:
  hostname: 'localhost'
  port: 5432
  username: 'postgres'
  user_database: 'helloprisma'
  password: ${PGPASSWORD}
  connectionTimeoutMillis: 3000
  user_dbclient: "prisma"
```

### Coding a Transaction with Prisma
Now, you can write an Operon application that uses Prisma to talk to the database.
If you've configured Operon to use Prisma, you can access the Prisma client from the [`.client`](../api-reference/contexts#transactionctxtclient) property of [`TransactionContext`](../api-reference/contexts#transactioncontextt) in Operon [transactions](./transaction-tutorial).
Here's an example of a transaction using Prisma to insert a row into the `operonhello` table:

```tsx
import { TransactionContext, HandlerContext, OperonTransaction, GetApi } from '@dbos-inc/operon';
import { PrismaClient } from "@prisma/client";

export class Hello {

  @OperonTransaction()
  static async helloTransaction(txnCtxt: TransactionContext<PrismaClient>, name: string)  {
    const greeting = `Hello, ${name}!`;
    console.log(greeting);
    const p: PrismaClient = txnCtxt.client as PrismaClient;
    const res = await p.operonHello.create({
        data: {
        greeting: greeting,
        },
    });
    return `Greeting ${res.greeting_id}: ${greeting}`;
  };

  @GetApi('/greeting/:name')
  static async helloHandler(handlerCtxt: HandlerContext, name: string) {
    return handlerCtxt.invoke(Hello).helloTransaction(name);
  }

}
```

The complete source code for this example is [here](https://github.com/dbos-inc/operon-demo-apps/tree/main/hello-prisma).