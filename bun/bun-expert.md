---
description: Expert Bun runtime development - the fast all-in-one JavaScript/TypeScript toolkit including runtime, bundler, test runner, and package manager
---

# Bun Expert

Expert guidance for building applications with Bun, the fast all-in-one JavaScript & TypeScript runtime. Covers runtime APIs, bundler, test runner, package manager, and migration from Node.js.

**Context7 Library ID**: `/llmstxt/bun_sh_llms_txt` - Use this to fetch latest documentation.

## Core Principles

### 1. Bun Runtime Fundamentals

**What is Bun?**
- All-in-one toolkit: runtime, bundler, test runner, package manager
- Built on JavaScriptCore (Safari's engine) - faster than V8
- Native TypeScript/JSX support without transpilation
- Drop-in Node.js replacement with high compatibility
- Written in Zig for maximum performance

**When to Use Bun:**
- New projects where speed matters
- Replacing Node.js for better performance
- Projects needing fast startup times (serverless, CLI tools)
- When you want fewer dependencies (built-in test runner, bundler)

### 2. HTTP Server with Bun.serve()

**Basic HTTP Server:**
```typescript
Bun.serve({
  port: 3000,
  fetch(req) {
    return new Response("Hello from Bun!");
  },
});
```

**With Routing:**
```typescript
Bun.serve({
  port: 3000,
  fetch(req) {
    const url = new URL(req.url);

    if (url.pathname === "/api/users") {
      return Response.json({ users: [] });
    }

    if (url.pathname === "/health") {
      return new Response("OK");
    }

    return new Response("Not Found", { status: 404 });
  },
});
```

**Static Routes (Zero-Allocation):**
```typescript
Bun.serve({
  routes: {
    "/health": new Response("OK"),
    "/ready": new Response("Ready", {
      headers: { "X-Ready": "1" },
    }),
    "/blog": Response.redirect("https://example.com/blog"),
    "/api/config": Response.json({
      version: "1.0.0",
      env: "production",
    }),
  },
});
```

**WebSocket Server:**
```typescript
Bun.serve({
  fetch(req, server) {
    if (server.upgrade(req)) {
      return; // WebSocket upgrade
    }
    return new Response("HTTP request");
  },
  websocket: {
    message(ws, message) {
      ws.send(`Echo: ${message}`);
    },
    open(ws) {
      console.log("WebSocket opened");
    },
    close(ws, code, reason) {
      console.log("WebSocket closed", code, reason);
    },
  },
});
```

### 3. File Operations

**Reading Files with Bun.file():**
```typescript
// Get file reference (lazy - doesn't read yet)
const file = Bun.file("data.json");

// Read as different formats
const text = await file.text();           // string
const json = await file.json();           // parsed JSON
const buffer = await file.arrayBuffer();  // ArrayBuffer
const stream = file.stream();             // ReadableStream

// Check if file exists
const exists = await file.exists();

// Get file metadata
console.log(file.size);  // size in bytes
console.log(file.type);  // MIME type
```

**Writing Files with Bun.write():**
```typescript
// Write string
await Bun.write("output.txt", "Hello World");

// Write JSON
await Bun.write("data.json", JSON.stringify({ key: "value" }));

// Write from Response
const response = await fetch("https://example.com/file");
await Bun.write("downloaded.txt", response);

// Copy file
await Bun.write("copy.txt", Bun.file("original.txt"));
```

**Streaming File Server:**
```typescript
Bun.serve({
  async fetch(req) {
    const path = new URL(req.url).pathname;
    const file = Bun.file(`./public${path}`);

    if (await file.exists()) {
      return new Response(file);
    }

    return new Response("Not Found", { status: 404 });
  },
});
```

### 4. SQLite with bun:sqlite

**Basic Database Operations:**
```typescript
import { Database } from "bun:sqlite";

// Open database (creates if doesn't exist)
const db = new Database("mydb.sqlite");

// In-memory database
const memDb = new Database(":memory:");

// Read-only mode
const readOnlyDb = new Database("mydb.sqlite", { readonly: true });
```

**Queries and Prepared Statements:**
```typescript
import { Database } from "bun:sqlite";

const db = new Database("app.sqlite");

// Create table
db.run(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT UNIQUE
  )
`);

// Insert with parameters (safe from SQL injection)
const insert = db.prepare("INSERT INTO users (name, email) VALUES (?, ?)");
insert.run("John", "john@example.com");

// Query single row
const user = db.prepare("SELECT * FROM users WHERE id = ?").get(1);

// Query multiple rows
const users = db.prepare("SELECT * FROM users").all();

// Named parameters
const query = db.prepare("SELECT * FROM users WHERE name = $name");
const result = query.all({ $name: "John" });
```

**Transactions:**
```typescript
const db = new Database("app.sqlite");

const insertMany = db.transaction((users) => {
  const insert = db.prepare("INSERT INTO users (name) VALUES (?)");
  for (const user of users) {
    insert.run(user.name);
  }
  return users.length;
});

const count = insertMany([
  { name: "Alice" },
  { name: "Bob" },
  { name: "Charlie" },
]);
```

### 5. Shell Commands with $

**Basic Usage:**
```typescript
import { $ } from "bun";

// Simple command
await $`echo "Hello World!"`;

// Capture output as text
const output = await $`ls -la`.text();

// Capture as buffer
const { stdout, stderr } = await $`echo "Hello"`.quiet();
```

**Variables and Interpolation:**
```typescript
import { $ } from "bun";

const name = "file.txt";
await $`cat ${name}`;  // Automatically escaped

// Environment variables
await $`echo $HOME`;

// Command substitution
await $`echo "Commit: $(git rev-parse HEAD)"`;
```

**Configuration:**
```typescript
import { $ } from "bun";

// Set global environment
$.env({ NODE_ENV: "production" });

// Set global working directory
$.cwd("/tmp");

// Override per-command
await $`pwd`.cwd("/home");
await $`echo $FOO`.env({ FOO: "bar" });
```

**Redirects and Pipes:**
```typescript
import { $ } from "bun";

// Redirect to file
await $`echo "Hello" > output.txt`;

// Append to file
await $`echo "World" >> output.txt`;

// Pipe commands
await $`cat file.txt | grep "pattern" | wc -l`;
```

### 6. Bundler

**Basic Build:**
```typescript
const result = await Bun.build({
  entrypoints: ["./src/index.ts"],
  outdir: "./dist",
});

if (!result.success) {
  console.error("Build failed:", result.logs);
}
```

**Build Configuration:**
```typescript
await Bun.build({
  entrypoints: ["./src/index.ts"],
  outdir: "./dist",

  // Target environment
  target: "browser",  // "browser" | "bun" | "node"

  // Output format
  format: "esm",  // "esm" | "cjs" | "iife"

  // Code splitting
  splitting: true,

  // Minification
  minify: true,
  // Or granular control:
  minify: {
    whitespace: true,
    syntax: true,
    identifiers: true,
  },

  // Sourcemaps
  sourcemap: "linked",  // "none" | "linked" | "inline" | "external"

  // Externalize packages
  external: ["react", "react-dom"],

  // Define globals
  define: {
    "process.env.NODE_ENV": JSON.stringify("production"),
  },

  // Custom loaders
  loader: {
    ".png": "file",
    ".svg": "text",
  },
});
```

**CLI Build:**
```bash
# Basic build
bun build ./src/index.ts --outdir ./dist

# With options
bun build ./src/index.ts --outdir ./dist --target browser --minify --sourcemap linked

# Multiple entrypoints
bun build ./src/index.ts ./src/worker.ts --outdir ./dist --splitting
```

### 7. Test Runner

**Writing Tests:**
```typescript
import { test, expect, describe, beforeAll, afterEach } from "bun:test";

describe("math", () => {
  test("addition", () => {
    expect(2 + 2).toBe(4);
  });

  test("async operation", async () => {
    const result = await Promise.resolve(42);
    expect(result).toBe(42);
  });
});
```

**Lifecycle Hooks:**
```typescript
import { test, expect, beforeAll, afterAll, beforeEach, afterEach } from "bun:test";

let db: Database;

beforeAll(() => {
  db = new Database(":memory:");
});

afterAll(() => {
  db.close();
});

beforeEach(() => {
  db.run("DELETE FROM users");
});

test("insert user", () => {
  // test code
});
```

**Matchers:**
```typescript
import { test, expect } from "bun:test";

test("matchers", () => {
  // Equality
  expect(value).toBe(expected);
  expect(value).toEqual(expected);  // Deep equality

  // Truthiness
  expect(value).toBeTruthy();
  expect(value).toBeFalsy();
  expect(value).toBeNull();
  expect(value).toBeUndefined();

  // Numbers
  expect(value).toBeGreaterThan(3);
  expect(value).toBeLessThanOrEqual(10);
  expect(value).toBeCloseTo(0.3, 5);

  // Strings
  expect(string).toMatch(/pattern/);
  expect(string).toContain("substring");

  // Arrays
  expect(array).toContain(item);
  expect(array).toHaveLength(3);

  // Objects
  expect(obj).toHaveProperty("key");
  expect(obj).toMatchObject({ key: "value" });

  // Exceptions
  expect(() => fn()).toThrow();
  expect(() => fn()).toThrow("error message");
});
```

**Snapshots:**
```typescript
import { test, expect } from "bun:test";

test("snapshot", () => {
  expect({ foo: "bar", count: 42 }).toMatchSnapshot();
});

// Update snapshots: bun test --update-snapshots
```

**Running Tests:**
```bash
# Run all tests
bun test

# Run specific file
bun test path/to/test.ts

# Run tests matching pattern
bun test --grep "user"

# Watch mode
bun test --watch

# Coverage
bun test --coverage

# JUnit output
bun test --reporter=junit --reporter-outfile=./results.xml
```

### 8. Package Manager

**Basic Commands:**
```bash
# Install all dependencies
bun install

# Add dependency
bun add zod

# Add dev dependency
bun add -d typescript

# Add global package
bun add -g prettier

# Remove package
bun remove lodash

# Update packages
bun update

# Run package binary
bunx prisma generate
```

**Workspaces (Monorepo):**
```json
// package.json
{
  "name": "my-monorepo",
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

**Workspace Dependencies:**
```json
// packages/pkg-a/package.json
{
  "name": "pkg-a",
  "dependencies": {
    "pkg-b": "workspace:*"
  }
}
```

**Filtered Operations:**
```bash
# Install for specific packages
bun install --filter "pkg-*"

# Run script in matching packages
bun --filter "lib-*" build
```

### 9. Environment Variables

**Automatic .env Loading:**
```typescript
// Bun automatically loads .env, .env.local, .env.development, etc.

// Access via Bun.env (preferred)
const apiKey = Bun.env.API_KEY;

// Or process.env (Node.js compatible)
const dbUrl = process.env.DATABASE_URL;
```

**Priority Order:**
1. `.env.${NODE_ENV}.local` (e.g., `.env.development.local`)
2. `.env.local`
3. `.env.${NODE_ENV}` (e.g., `.env.development`)
4. `.env`

### 10. TypeScript Support

**Native TypeScript:**
```typescript
// No configuration needed - just run it
// bun run app.ts

// Type annotations work out of the box
function greet(name: string): string {
  return `Hello, ${name}!`;
}
```

**tsconfig.json Integration:**
```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "types": ["bun-types"],
    "strict": true
  }
}
```

**Path Aliases:**
```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

```typescript
// Works automatically
import { helper } from "@/utils/helper";
```

## Performance Tips

### 1. Use Bun-Native APIs
```typescript
// Prefer Bun.file() over fs
const content = await Bun.file("data.txt").text();

// Prefer Bun.write() over fs.writeFile
await Bun.write("output.txt", content);

// Prefer Bun.serve() over Express/Fastify
Bun.serve({ fetch: (req) => new Response("OK") });
```

### 2. Use bun:sqlite for Databases
```typescript
// Much faster than better-sqlite3 or node-sqlite3
import { Database } from "bun:sqlite";
const db = new Database("app.db");
```

### 3. Leverage Streaming
```typescript
// Stream large files instead of loading into memory
Bun.serve({
  fetch(req) {
    return new Response(Bun.file("large-video.mp4"));
  },
});
```

### 4. Use Prepared Statements
```typescript
// Prepare once, execute many times
const stmt = db.prepare("SELECT * FROM users WHERE id = ?");

for (const id of userIds) {
  const user = stmt.get(id);
}
```

## Node.js Migration Guide

### Package.json Scripts
```json
{
  "scripts": {
    "start": "bun run src/index.ts",
    "dev": "bun --watch src/index.ts",
    "test": "bun test",
    "build": "bun build src/index.ts --outdir dist"
  }
}
```

### Common Replacements
| Node.js | Bun |
|---------|-----|
| `node app.js` | `bun app.js` or `bun app.ts` |
| `npm install` | `bun install` |
| `npx` | `bunx` |
| `npm run` | `bun run` |
| `jest` / `vitest` | `bun test` |
| `webpack` / `esbuild` | `bun build` |
| `nodemon` | `bun --watch` |

### API Compatibility
Most Node.js APIs work in Bun:
- `fs`, `path`, `os`, `crypto` - Full support
- `http`, `https` - Supported (but prefer `Bun.serve()`)
- `child_process` - Supported (but prefer `Bun.spawn()` or `$`)
- `worker_threads` - Supported
- `stream` - Supported

## Quick Reference

### CLI Commands
```bash
bun run <file>          # Run a file
bun --watch <file>      # Run with auto-reload
bun install             # Install dependencies
bun add <pkg>           # Add package
bun remove <pkg>        # Remove package
bun test                # Run tests
bun build               # Bundle code
bunx <pkg>              # Run package without installing
bun upgrade             # Upgrade Bun itself
```

### Key APIs
```typescript
// HTTP Server
Bun.serve({ fetch, port })

// File I/O
Bun.file(path)
Bun.write(path, content)

// Shell
import { $ } from "bun"
await $`command`

// SQLite
import { Database } from "bun:sqlite"

// Testing
import { test, expect } from "bun:test"

// Process
Bun.spawn(["cmd", "args"])
Bun.env.VARIABLE

// Utilities
Bun.sleep(ms)
Bun.hash(data)
Bun.password.hash(password)
```
