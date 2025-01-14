---
sidebar_position: 2
title: Configuration
description: DBOS configuration reference
---

You can configure the DBOS Transact runtime with a configuration file.
Valid configuration files must be:
- Named `dbos-config.yaml`
- Located at the application package root.
- Valid [YAML](https://yaml.org/) conforming to the schema described below.

::::info
You can use environment variables for configuration values through the syntax `key: ${VALUE}`.
We strongly recommend using an environment variable for the database password field, as demonstrated below.
::::

---

### Database

The database section is used to set up the connection to the database.
DBOS currently only supports Postgres-compatible databases.
Every field is required unless otherwise specified.

- **hostname**: Database server hostname. For local deployment only, not used in DBOS Cloud.
- **port**: Database server port. For local deployment only, not used in DBOS Cloud.
- **username**: Username with which to connect to the database server. For local deployment only, not used in DBOS Cloud.
- **password**: Password with which to connect to the database server.  We recommend using an environment variable for this field, instead of plain text. For local deployment only, not used in DBOS Cloud.
- **app_db_name**: Name of the application database.
- **sys_db_name** (optional): Name of the system database in which DBOS stores internal state. Defaults to `{app_db_name}_dbos_sys`.  For local deployment only, not used in DBOS Cloud.
- **app_db_client** (optional): Client to use for connecting to the application database. Must be one of `knex`, `typeorm`, or `prisma`.  Defaults to `knex`.  The client specified here is the one used in [`TransactionContext`](../api-reference/contexts#transactioncontextt).
- **ssl_ca** (optional): If using SSL/TLS to securely connect to a database, path to an SSL root certificate file.  Equivalent to the [`sslrootcert`](https://www.postgresql.org/docs/current/libpq-ssl.html) connection parameter in `psql`.
- **connectionTimeoutMillis** (optional): Database connection timeout in milliseconds. Defaults to `3000`.
- **migrate** (optional): A list of commands to run to apply your application's schema to the database. We recommend using a migration tool like those built into [Knex](https://knexjs.org/guide/migrations.html), [TypeORM](https://typeorm.io/migrations), and [Prisma](https://www.prisma.io/docs/orm/prisma-migrate).
- **rollback** (optional) A list of commands to run to roll back the last batch of schema migrations.

**Example**:

```yaml
database:
  hostname: 'localhost'
  port: 5432
  username: 'postgres'
  password: ${PGPASSWORD}
  app_db_name: 'hello'
  app_db_client: 'knex'
  migrate: ['npx knex migrate:latest']
  rollback: ['npx knex migrate:rollback']
```

::::info
The role with which DBOS connects to Postgres must have [`CREATE`](https://www.postgresql.org/docs/current/ddl-priv.html#DDL-PRIV-CREATE) privileges on both the application database and system database if they already exist.
If either does not exist, the Postgres role must have the [`CREATEDB`](https://www.postgresql.org/docs/current/sql-createdatabase.html) privilege to create them.
::::

---

### Runtime

This section is used to specify DBOS runtime parameters.

- **port** (optional): The port from which to serve your functions. Defaults to `3000`. Using [`npx dbos start -p <port>`](./cli#npx-dbos-start) overrides this config parameter.
- **entrypoints** (optional): The compiled Javascript files where DBOS looks for your application's code. At startup, the DBOS runtime automatically loads all classes exported from these files, serving their endpoints and registering their decorated functions. Defaults to `[dist/operations.js]`.

**Example**:

```yaml
runtimeConfig:
  port: 3000 # (default)
  entrypoints:
    - 'dist/operations.js' # (default)
```
---

### HTTP

This section is used to specify DBOS HTTP parameters.

- **cors_middleware** (optional): Whether to allow cross-origin requests (via CORS middleware).  Default is `true`.
- **credentials** (optional): If `cors_middleware` is enabled, whether to allow cross-origin requests that include credentials.  Default is `true`.
- **allowed_origins** (optional): If `cors_middleware` is enabled, a list of origins/domains that are permitted to make cross-origin requests.  Default is to allow any origin.

**Example**:

```yaml
http:
  cors_middleware: true
  credentials: true
  allowed_origins:
    - 'https://partner.com'
    - 'https://app.internal.com'
```
---

### Application

Applications can optionally use the application configuration to define custom properties as key-value pairs.
These properties can be retrieved from any [context](./contexts) via the [`getConfig`](../api-reference/contexts#ctxtgetconfig) method.

**Example**:
```yaml
application:
  PAYMENTS_SERVICE: 'https://stripe.com/payment'
```

### Environment Variables

Applications can optionally use the `env` configuration to define environment variables.
These are set in your application before its code is initialized and can be retrieved from `process.env` like any other environment variables.
For example, the `WEB_PORTAL` variable set below could be retrieved from an application as `process.env.WEB_PORTAL`.
Environment variables configured here are automatically exported to and set in DBOS Cloud.

**Example**:
```yaml
env:
  WEB_PORTAL: 'https://example.com'
```

---

### Telemetry

You can use the configuration file to tune the behavior of DBOS logging facility.
Note all options in this section are optional and will, if not specified, use the default values indicated in the example below.

#### Logs
- **logLevel**: Filters, by severity, what logs should be printed. Defaults to `'info'`. Using [`npx dbos start -l <logLevel>`](./cli#npx-dbos-start) overrides this config parameter.
- **addContextMetadata**: Enables the addition of contextual information, such as workflow identity UUID, to each log entry. Defaults to `true`.
- **silent**: Silences the logger. Defaults to `false`.

**Example**:

```yaml
telemetry:
  logs:
    logLevel: 'info' # info (default) | debug | warn | emerg | alert | crit | error
    addContextMetadata: true # true (default) | false
    silent: false # false (default) | true
```

--- 

### Configuration Schema File

There is a schema file available for the DBOS configuration file schema [in our GitHub repo](https://raw.githubusercontent.com/dbos-inc/dbos-ts/main/dbos-config.schema.json).
This schema file can be used to provide an improved YAML editing experience for developer tools that leverage it.
For example, the Visual Studio Code [RedHat YAML extension](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) provides tooltips, statement completion and real-time validation for editing DBOS config files. 
This extension provides [multiple ways](https://github.com/redhat-developer/vscode-yaml#associating-schemas) to associate a YAML file with its schema.
The easiest is to simply add a comment with a link to the schema at the top of the config file:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/dbos-inc/dbos-ts/main/dbos-config.schema.json
```
