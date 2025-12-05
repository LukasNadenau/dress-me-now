### Installing Supawright with pnpm

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Command to install the Supawright library as a development dependency using the npm package manager.

```bash
npm i -D supawright
```

--------------------------------

### Example Database Schema (SQL)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

SQL DDL for example `user` and `session` tables in the `public` schema, illustrating a foreign key relationship between `session.user_id` and `user.id`. This schema is used to demonstrate Supawright's recursive data handling.

```sql
create table public."user" (
    id uuid primary key default uuid_generate_v4(),
    email text not null unique,
    password text not null
);

create table public.session (
    id uuid primary key default uuid_generate_v4(),
    user_id uuid not null references public."user"(id),
    token text,
    created_at timestamp with time zone not null default now()
);
```

--------------------------------

### Automating Supabase Type Fix with Make and sed

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Example of a Make target that generates Supabase TypeScript types locally and uses `sed` to replace `export interface Database {` with `export type Database = {`, saving the output to a specified file.

```make
types:
	pnpm supabase gen types typescript --local | \
	sed 's/export interface Database {/export type Database = {/' \
	> src/types/database.ts
```

--------------------------------

### Setting up Playwright Test with Supawright (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Demonstrates how to wrap the Playwright `test` function with `withSupawright`, providing the database type and specifying the schemas Supawright should manage. Requires importing `withSupawright` and the database types.

```typescript
import { withSupawright } from 'supawright'
import type { Database } from './database'

const test = withSupawright<
  Database,
  'public' | 'other' // Note 1
>(['public', 'other'])
```

--------------------------------

### Creating a Record with Supawright (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Shows how to use the `supawright.create` fixture to create a database record. When creating a `session`, Supawright automatically creates the related `user` record due to the foreign key constraint and populates the `user_id`.

```typescript
test('can login', async ({ supawright }) => {
  const session = await supawright.create('public', 'session')
  expect(session.user_id).toBeDefined()
})
```

--------------------------------

### Overriding Supawright Connection Details (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Shows how to explicitly configure the Supabase URL, service role key, and the database connection details (host, port, user, password, database) when initializing `withSupawright`, instead of relying on environment variables.

```TypeScript
const test = withSupawright<Database, 'public' | 'other'>(['public', 'other'], {
  supabase: {
    supabaseUrl: 'my-supabase-url.com',
    serviceRoleKey: 'my-service-role-key'
  },
  database: {
    host: 'localhost',
    port: 54322,
    user: 'me',
    password: 'password',
    database: 'my-database'
  }
})
```

--------------------------------

### Running Code Before Supawright Teardown (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Illustrates how to provide a function to `beforeTeardown` that executes custom logic after each test but before Supawright cleans up created records. Useful for tasks like waiting for network states.

```TypeScript
const test = withSupawright<Database, 'public' | 'other'>(['public', 'other'], {
  beforeTeardown: async ({ page, supawright }) => {
    await page.waitForLoadState('networkidle')
  }
})
```

--------------------------------

### Configuring Supawright with Custom Generators and Overrides (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Shows how to provide custom functions for generating fake data (`generators`) and overriding the default record creation logic (`overrides`) when using `withSupawright`. Useful for handling specific data types or triggering database logic.

```TypeScript
const test = withSupawright<
    Database,
    'public' | 'other',
>(
    ['public', 'other'],
    {
        generators: {
            smallint: () => 123,
            text: (table: string, column: string) => `${table}.${column}`,
        },
        overrides: {
            public: {
                user: async ({ supawright, data, supabase, generators }) => {
                    const { data: user } = await supabase
                        .from('user')
                        .insert(...)
                        .select()
                        .single()
                    ...
                    return [{
                        schema: 'public',
                        table: 'user',
                        data: user,
                    }]
                }
            }
        }
    }
)
```

--------------------------------

### Creating Records with Custom Data and FK Override (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Illustrates passing a data object to `supawright.create` to set specific column values. When a foreign key column (`user_id`) is provided, Supawright skips creating the related record (`user`), as verified by querying the database directly using `supawright.supabase()`.

```typescript
test('can login', async ({ supawright }) => {
  const user = await supawright.create('user', {
    email: 'some-email@supawrightmail.com'
  })
  const session = await supawright.create('session', {
    user_id: user.id
  })
  // Supawright will not create a user record, since we've passed in
  // a user_id.
  const { data: users } = await supawright.supabase().from('user').select()
  expect(users.length).toBe(1)
})
```

--------------------------------

### Automatic Cleanup of Related Records (TypeScript)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Shows how Supawright automatically discovers and cleans up records created outside of its `create` method, provided they have a foreign key relationship with a record Supawright *did* create. This is demonstrated by creating `session` records using the standard Supabase client after creating a `user` with Supawright.

```typescript
test('can login', async ({ supawright }) => {
  const user = await supawright.create('user')

  // Since we're using the standard Supabase client here, Supawright
  // is unaware of the records we're creating.
  await supawright
    .supabase()
    .from('session')
    .insert([{ user_id: user.id }, { user_id: user.id }])

  // However, Supawright will discover these records and delete
  // them when the test exits.
})
```

--------------------------------

### Fixing Supabase Generated Types (interface to type)

Source: https://github.com/isaacharrisholt/supawright/blob/main/README.md

Shows the required change in the generated Supabase types file (typically `database.ts`) to change the `Database` definition from an `interface` to a `type` for compatibility with Supawright's type constraints.

```diff
- export interface Database {
+ export type Database = {
```

=== COMPLETE CONTENT === This response contains all available snippets from this library. No additional content exists. Do not make further requests.