# typescript-for-professionals

TypeScript Mastery Notebook (Project-Based)
Target: Experienced software engineers (1+ years)
Goal: Production-grade TypeScript expertise across the full stack
Format: Obsidian‑ready Markdown (use in Notion, VS Code, GitHub, Jupyter)

## Table of Contents
- [Environment Setup](#environment-setup)

- [TypeScript Type System Mastery](#typescript-type-system-mastery)

- [Testing TypeScript](#testing-typescript)

- [TypeScript in Node.js – REST API](#typescript-in-nodejs-rest-api)

- [TypeScript in React](#typescript-in-react)

- [TypeScript in Next.js](#typescript-in-nextjs)

- [TypeScript and Databases](#typescript-and-databases)

- [TypeScript Design Patterns](#typescript-design-patterns)

- [TypeScript Architecture](#typescript-architecture)

- [Enterprise Monorepos & Workspaces](#enterprise-monorepos-workspaces)

- [Modern Developer Experience (DX)](#modern-developer-experience-dx)

- [Production Backend & NestJS](#production-backend-nestjs)

- [CI/CD, Docker & Deployment](#cicd-docker-deployment)

- [TypeScript Best Practices](#typescript-best-practices)

- [Real Projects](#real-projects)

- [Final Resources](#final-resources)

## Environment Setup
### 1.1 Installing TypeScript
#### Node.js
Install via nvm (macOS/Linux) or nvm-windows.

Use the latest LTS (e.g., 20.x).

#### Package Managers
Choose one:

```bash
# npm (default)
npm install typescript --save-dev

# pnpm
pnpm add typescript -D

# yarn
yarn add typescript --dev
```
Verify
```bash
npx tsc --version
```
### 1.2 Project Initialisation
```bash
mkdir my-ts-project
cd my-ts-project
npm init -y
```
Then add TypeScript:

```bash
npm install typescript --save-dev
npx tsc --init
```
### 1.3 tsconfig.json – Production‑ready Configuration
```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    
    /* Emit */
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    
    /* Interop Constraints */
    "isolatedModules": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    
    /* Type Checking */
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    
    /* Projects & Paths */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "skipLibCheck": true
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules", "dist"]
}
```
Explanation of key options:

strict: enables all strict type‑checking options.

noUncheckedIndexedAccess: forces undefined checks when accessing arrays/objects by index.

exactOptionalPropertyTypes: prevents assigning `undefined` to optional properties incorrectly.

isolatedModules: ensures each file can be safely transpiled without relying on other files (required for tools like esbuild, SWC, and Vite).

module: "NodeNext" + moduleResolution: "NodeNext": full ESM/CJS compatibility.

paths: enables clean imports like import { x } from '@/utils'.

skipLibCheck: speeds up compilation by skipping type checks of .d.ts files.

Running TypeScript
### 2.1 Compilation
```bash
npx tsc          # compiles once
npx tsc --watch  # watch mode
```
### 2.2 Development Runners
ts-node is largely outdated for modern ESM modules. Use `tsx` (modern, recommended).

```bash
npm install tsx --save-dev
npx tsx src/index.ts       # run once
npx tsx watch src/index.ts # hot reload
```

### 2.3 Build & Production
Relying on `tsc` for building is slow. Use a modern bundler like `tsup` for Node.js backends or `Vite` for frontends, and use `tsc` just for type-checking.

```bash
npm install tsup --save-dev

package.json scripts:

```
```json
{
  "scripts": {
    "build": "tsup src/index.ts --format esm,cjs --dts",
    "typecheck": "tsc --noEmit",
    "start": "node dist/index.js",
    "dev": "tsx watch src/index.ts"
  }
}
```

Production build:

```bash
npm run build
npm start
```
## TypeScript Type System Mastery
We assume you already know basic annotations. This section consolidates the advanced type system features you’ll use daily.

### 3.1 Utility Types (Built‑in)
```typescript
type User = { id: number; name: string; email?: string };

type PartialUser = Partial<User>;          // all optional
type RequiredUser = Required<User>;        // all required
type ReadonlyUser = Readonly<User>;        // all readonly
type UserContact = Pick<User, 'email' | 'name'>; // subset
type UserWithoutId = Omit<User, 'id'>;     // remove keys
type UserRecord = Record<'id' | 'name', string>; // object from keys
```
### 3.2 Mapped Types & Template Literal Types
```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getName: () => string; getEmail: () => string | undefined; getId: () => number }

// Deep Readonly (recursive)
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```
### 3.3 Conditional Types with infer
```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type Awaited<T> = T extends Promise<infer U> ? Awaited<U> : T;

// Unboxing types from arrays / promises
type ElementType<T> = T extends (infer U)[] ? U : never;
```
### 3.4 Generic Constraints & extends
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Generic repository with constraint
interface Repository<T extends { id: string }> {
  findById(id: string): T | undefined;
  save(item: T): void;
}
```
### 3.5 Discriminated Unions for Exhaustive Checking
```typescript
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'rectangle'; width: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':    return Math.PI * shape.radius ** 2;
    case 'rectangle': return shape.width * shape.height;
    default:          const _exhaustive: never = shape; return _exhaustive;
  }
}
```
### 3.6 Branded Types (Nominal Typing)
```typescript
type UserId = string & { __brand: 'UserId' };
type Email = string & { __brand: 'Email' };

function createUser(id: UserId, email: Email) { ... }

const userId = 'abc' as UserId;
const email = 'a@b.com' as Email;
```
### 3.7 Advanced Type Safety & Narrowing
Type Guards (`is`):
```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string';
}
```

Assertion Functions (`asserts`):
```typescript
function assertIsDefined<T>(val: T): asserts val is NonNullable<T> {
  if (val === undefined || val === null) {
    throw new Error(`Expected 'val' to be defined, but received ${val}`);
  }
}
```

The `satisfies` operator:
```typescript
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

// Validates keys and values without losing the specific shape of the object
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255]
} satisfies Record<Colors, string | RGB>;
```

`import type`:
Always use `import type` when only importing types. This helps bundlers like esbuild or swc optimize code by fully stripping type imports.

```typescript
import type { User } from './types';
import { createUser } from './utils';
```

### 3.8 Common Mistakes & Best Practices
Mistake: Using any → Fix: Use unknown + validation.

Mistake: Overly deep recursive types causing compiler slowdown → Fix: Use lazy evaluation or interface.

Mistake: Forgetting readonly for immutable data → Fix: as const and Readonly<T>.

Best: Validate all external inputs with Zod (covered later).

## Testing TypeScript
### 4.1 Jest Setup
```bash
npm install -D jest @types/jest ts-jest
npx ts-jest config:init
jest.config.ts:

```
```typescript
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
};
```
Example unit test:

```typescript
import { add } from './math';

test('adds numbers', () => {
  expect(add(1, 2)).toBe(3);
});
```
### 4.2 Vitest (Modern, Faster)
```bash
npm install -D vitest
```
vitest.config.ts:

```typescript
import { defineConfig } from 'vitest/config';
export default defineConfig({
  test: { globals: true },
});
```
### 4.3 React Testing Library
```bash
npm install -D @testing-library/react @testing-library/jest-dom jsdom
```
vitest.config.ts add:

```typescript
environment: 'jsdom',
setupFiles: './src/test-setup.ts',
```
Example:

```typescript
import { render, screen } from '@testing-library/react';
import Button from './Button';

test('renders button', () => {
  render(<Button label="Click" />);
  expect(screen.getByText('Click')).toBeInTheDocument();
});
```

### 4.4 Mocking with MSW (Mock Service Worker)
MSW intercepts network requests at the network level for robust frontend testing without hitting real APIs.

```bash
npm install -D msw

```
```typescript
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

export const handlers = [
  http.get('/api/user', () => {
    return HttpResponse.json({ name: 'John Doe' });
  }),
];

const server = setupServer(...handlers);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### 4.5 API / Integration Testing
Use supertest:

```typescript
import request from 'supertest';
import app from './app';

it('GET /health returns 200', async () => {
  const res = await request(app).get('/health');
  expect(res.status).toBe(200);
  expect(res.body).toEqual({ status: 'ok' });
});
```
### 4.6 E2E Testing (Playwright)
```bash
npm init playwright@latest
```
```typescript
import { test, expect } from '@playwright/test';

test('homepage has title', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveTitle('My App');
});
```
## TypeScript in Node.js – REST API
Build a production‑ready Express API with full type safety.

### 5.1 Project Setup
```bash
mkdir node-api && cd node-api && npm init -y
npm install express cors helmet
npm install -D typescript @types/express @types/cors @types/node tsx zod prisma @prisma/client
npx tsc --init (use the tsconfig from Section 1.3)
npx prisma init --datasource-provider postgresql
```
### 5.2 Architecture
```text
src/
├── index.ts
├── app.ts
├── config.ts          (env vars parsed by Zod)
├── middleware/
│   ├── errorHandler.ts
│   └── auth.ts
├── modules/
│   ├── user/
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   └── user.types.ts
│   └── ...
├── shared/
│   ├── types.ts
│   └── utils.ts
└── prisma/
    └── schema.prisma
```
### 5.3 Environment Configuration
src/config.ts:

```typescript
import { z } from 'zod';

const envSchema = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
});

export const config = envSchema.parse(process.env);
```
### 5.4 Express App Factory
src/app.ts:

```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { errorHandler } from './middleware/errorHandler';
import userRouter from './modules/user/user.routes';

export function createApp() {
  const app = express();
  app.use(helmet());
  app.use(cors());
  app.use(express.json());
  app.use('/api/users', userRouter);
  app.use(errorHandler);
  return app;
}
```
### 5.5 Base Repository & User Implementation
src/shared/types.ts:

```typescript
export interface Repository<T extends { id: string }> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(data: Omit<T, 'id'>): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}
src/modules/user/user.types.ts:

```
```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  passwordHash: string;
  role: 'user' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}
src/modules/user/user.repository.ts (using Prisma):

```
```typescript
import { PrismaClient } from '@prisma/client';
import { User } from './user.types';

export class UserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  async create(data: Omit<User, 'id' | 'createdAt' | 'updatedAt'>): Promise<User> {
    return this.prisma.user.create({ data });
  }
}
```
### 5.6 Service Layer with Business Logic
src/modules/user/user.service.ts:

```typescript
import { UserRepository } from './user.repository';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { config } from '../../config';

export class UserService {
  constructor(private repo: UserRepository) {}

  async register(email: string, password: string, name: string) {
    const existing = await this.repo.findByEmail(email);
    if (existing) throw new AppError(409, 'Email already registered');
    const passwordHash = await bcrypt.hash(password, 12);
    const user = await this.repo.create({ email, name, passwordHash, role: 'user' });
    return this.generateToken(user);
  }

  async login(email: string, password: string) {
    const user = await this.repo.findByEmail(email);
    if (!user) throw new AppError(401, 'Invalid credentials');
    const valid = await bcrypt.compare(password, user.passwordHash);
    if (!valid) throw new AppError(401, 'Invalid credentials');
    return this.generateToken(user);
  }

  private generateToken(user: { id: string; role: string }) {
    return jwt.sign({ sub: user.id, role: user.role }, config.JWT_SECRET, { expiresIn: '1h' });
  }
}
```
### 5.7 Controller & Validation (Zod)
src/modules/user/user.controller.ts:

```typescript
import { Request, Response, NextFunction } from 'express';
import { UserService } from './user.service';
import { z } from 'zod';

const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(1),
});

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string(),
});

export class UserController {
  constructor(private service: UserService) {}

  register = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const data = registerSchema.parse(req.body);
      const token = await this.service.register(data.email, data.password, data.name);
      res.status(201).json({ token });
    } catch (err) {
      next(err);
    }
  };

  login = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const data = loginSchema.parse(req.body);
      const token = await this.service.login(data.email, data.password);
      res.json({ token });
    } catch (err) {
      next(err);
    }
  };
}
```
### 5.8 Routes & Dependency Injection
src/modules/user/user.routes.ts:

```typescript
import { Router } from 'express';
import { PrismaClient } from '@prisma/client';
import { UserRepository } from './user.repository';
import { UserService } from './user.service';
import { UserController } from './user.controller';

const prisma = new PrismaClient();
const repo = new UserRepository(prisma);
const service = new UserService(repo);
const controller = new UserController(service);

const router = Router();
router.post('/register', controller.register);
router.post('/login', controller.login);
export default router;
```
### 5.9 Error Handling Middleware
src/middleware/errorHandler.ts:

```typescript
import { Request, Response, NextFunction } from 'express';

export class AppError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}

export function errorHandler(err: Error, _req: Request, res: Response, _next: NextFunction) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({ error: err.message });
  }
  console.error(err);
  res.status(500).json({ error: 'Internal Server Error' });
}
```
### 5.10 Entry Point
src/index.ts:

```typescript
import { createApp } from './app';
import { config } from './config';

const app = createApp();

app.listen(config.PORT, () => {
  console.log(`Server running on port ${config.PORT}`);
});
```
### 5.11 Testing the API
```typescript
import supertest from 'supertest';
import { createApp } from './app';

const app = createApp();

describe('User endpoints', () => {
  it('POST /register returns token', async () => {
    const res = await supertest(app)
      .post('/api/users/register')
      .send({ email: 'test@example.com', password: '12345678', name: 'Test' });
    expect(res.status).toBe(201);
    expect(res.body.token).toBeDefined();
  });
});
```
## TypeScript in React
### 6.1 Component Typing
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary';
  label: string;
  onClick: () => void;
  disabled?: boolean;
}

// Modern React discourages React.FC. Use standard function signatures.
function Button({ variant, label, onClick, disabled = false }: ButtonProps) {
  return (
    <button className={`btn-${variant}`} onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}
```
### 6.2 Hooks with Type Inference
```typescript
import { useState, useReducer, useRef } from 'react';

// useState
const [count, setCount] = useState<number>(0);

// useReducer
type Action = { type: 'increment' } | { type: 'decrement' };
const reducer = (state: number, action: Action): number => {
  switch (action.type) {
    case 'increment': return state + 1;
    case 'decrement': return state - 1;
  }
};
const [state, dispatch] = useReducer(reducer, 0);

// useRef
const inputRef = useRef<HTMLInputElement>(null);
```
### 6.3 Custom Hooks
```typescript
import { useState, useEffect } from 'react';

function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then((json: T) => setData(json))
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// usage: const { data: users } = useFetch<User[]>('/api/users');
```
### 6.4 Context with TypeScript
```typescript
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggle: () => void;
}

const ThemeContext = React.createContext<ThemeContextType | undefined>(undefined);

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
};
```
### 6.5 Forms with react-hook-form + Zod
```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});
type FormData = z.infer<typeof schema>;

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormData) => console.log(data);
  // ...
}
```
### 6.6 React Query (TanStack Query)
```typescript
import { useQuery, useMutation } from '@tanstack/react-query';

function useUsers() {
  return useQuery<User[], Error>({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(res => res.json()),
  });
}

function useCreateUser() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (newUser: Omit<User, 'id'>) =>
      fetch('/api/users', { method: 'POST', body: JSON.stringify(newUser) }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['users'] }),
  });
}
```
### 6.7 Example: Full Dashboard Page
```typescript
export default function Dashboard() {
  const { data: users, isLoading, error } = useUsers();
  if (isLoading) return <Spinner />;
  if (error) return <ErrorAlert message={error.message} />;
  return <UserTable users={users!} />;
}
```
## TypeScript in Next.js
### 7.1 App Router Setup
```text
app/
├── layout.tsx
├── page.tsx
├── dashboard/
│   └── page.tsx
├── api/
│   ├── users/route.ts
└── actions.ts
```
### 7.2 Server Components
```typescript
// app/page.tsx
import { db } from '@/lib/db';

export default async function HomePage() {
  const posts = await db.post.findMany({ take: 10 });
  return (
    <main>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.content}</p>
        </article>
      ))}
    </main>
  );
}
```
### 7.3 Route Handlers (API)
```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

export async function POST(req: NextRequest) {
  const body = await req.json();
  const parsed = createUserSchema.safeParse(body);
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 });
  }
  const user = await db.user.create({ data: parsed.data });
  return NextResponse.json(user, { status: 201 });
}
```
### 7.4 Server Actions
```typescript
// app/actions.ts
'use server';
import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';
import { z } from 'zod';

const schema = z.object({ title: z.string(), content: z.string() });

export async function createPost(formData: FormData) {
  const data = schema.parse(Object.fromEntries(formData));
  await db.post.create({ data });
  revalidatePath('/');
}
```
Usage in a form:

```typescript
import { createPost } from '@/app/actions';

export function NewPostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="content" />
      <button type="submit">Create</button>
    </form>
  );
}
```
### 7.5 Middleware & Authentication
```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { jwtVerify } from 'jose';

export async function middleware(req: NextRequest) {
  const token = req.cookies.get('token')?.value;
  if (!token) return NextResponse.redirect(new URL('/login', req.url));
  try {
    const { payload } = await jwtVerify(token, new TextEncoder().encode(process.env.JWT_SECRET));
    // Attach user to request (via headers or a custom property)
    req.headers.set('x-user-id', payload.sub as string);
    return NextResponse.next();
  } catch {
    return NextResponse.redirect(new URL('/login', req.url));
  }
}

export const config = { matcher: ['/dashboard/:path*'] };
```
## TypeScript and Databases
### 8.1 Prisma + PostgreSQL
Schema (prisma/schema.prisma):

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  password  String
  role      String   @default("user")
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
}
```
Using the generated client:

```typescript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function getUsersWithPosts() {
  return prisma.user.findMany({ include: { posts: true } });
}
// Type is automatically (User & { posts: Post[] })[]
```
### 8.2 Drizzle ORM (Type‑Safe SQL)
```typescript
import { pgTable, serial, text, varchar } from 'drizzle-orm/pg-core';
import { drizzle } from 'drizzle-orm/node-postgres';
import { eq } from 'drizzle-orm';

const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email').notNull().unique(),
  name: text('name').notNull(),
});

const db = drizzle(process.env.DATABASE_URL!);

async function getUserById(id: number) {
  return db.select().from(users).where(eq(users.id, id));
}
```
### 8.3 Mongoose (MongoDB)
```typescript
import mongoose, { Schema, InferSchemaType } from 'mongoose';

const UserSchema = new Schema({
  email: { type: String, required: true, unique: true },
  name:  { type: String, required: true },
});

// Modern Mongoose (v6+) natively infers types from schemas
type UserType = InferSchemaType<typeof UserSchema>;

const User = mongoose.model('User', UserSchema);
```
## TypeScript Design Patterns
### 9.1 Singleton
```typescript
class Config {
  private static instance: Config;
  private constructor(public readonly env: string) {}
  static getInstance(): Config {
    if (!Config.instance) Config.instance = new Config(process.env.NODE_ENV ?? 'dev');
    return Config.instance;
  }
}
```
### 9.2 Factory
```typescript
interface Logger { log(msg: string): void; }
class ConsoleLogger implements Logger { log(msg: string) { console.log(msg); } }
class FileLogger implements Logger { log(msg: string) { /* write to file */ } }

function createLogger(type: 'console' | 'file'): Logger {
  if (type === 'console') return new ConsoleLogger();
  return new FileLogger();
}
```
### 9.3 Repository (Generic)
```typescript
interface ReadRepository<T, ID> {
  findById(id: ID): Promise<T | null>;
}

interface WriteRepository<T, ID> {
  save(entity: T): Promise<void>;
  delete(id: ID): Promise<void>;
}

export type Repository<T, ID> = ReadRepository<T, ID> & WriteRepository<T, ID>;
```
### 9.4 Strategy
```typescript
interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
  pay(amount: number) { /* ... */ }
}

class PayPalPayment implements PaymentStrategy {
  pay(amount: number) { /* ... */ }
}

class Checkout {
  constructor(private strategy: PaymentStrategy) {}
  executePayment(amount: number) { this.strategy.pay(amount); }
}
```
### 9.5 Observer / Event Emitter
```typescript
type Listener<T extends unknown[]> = (...args: T) => void;

class TypedEmitter<Events extends Record<string, unknown[]>> {
  private listeners: { [K in keyof Events]?: Listener<Events[K]>[] } = {};

  on<K extends keyof Events>(event: K, listener: Listener<Events[K]>) {
    (this.listeners[event] ??= []).push(listener);
  }

  emit<K extends keyof Events>(event: K, ...args: Events[K]) {
    this.listeners[event]?.forEach(l => l(...args));
  }
}
```
## TypeScript Architecture
### 10.1 Clean Architecture (Example)
```text
src/
├── domain/         (entities, value objects, interfaces)
├── application/    (use cases, DTOs)
├── infrastructure/ (repositories, external services)
└── presentation/   (controllers, routes)
```
Domain:

```typescript
// domain/User.ts
export interface User {
  id: string;
  email: string;
  password: string;
}

export interface IUserRepository {
  findByEmail(email: string): Promise<User | null>;
  save(user: User): Promise<void>;
}
```
Application:

```typescript
// application/RegisterUseCase.ts
export class RegisterUseCase {
  constructor(private userRepo: IUserRepository, private hasher: IPasswordHasher) {}
  async execute(email: string, password: string): Promise<User> {
    const existing = await this.userRepo.findByEmail(email);
    if (existing) throw new Error('Exists');
    const hashed = await this.hasher.hash(password);
    const user = { id: uuid(), email, password: hashed };
    await this.userRepo.save(user);
    return user;
  }
}
```
Infrastructure:

```typescript
// infrastructure/PrismaUserRepo.ts
export class PrismaUserRepo implements IUserRepository {
  constructor(private prisma: PrismaClient) {}
  async findByEmail(email: string) { return this.prisma.user.findUnique({ where: { email } }); }
  async save(user: User) { await this.prisma.user.create({ data: user }); }
}
```
### 10.2 Feature‑Based Architecture
```text
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types.ts
│   ├── dashboard/
│   └── ...
├── shared/
│   ├── ui/
│   ├── lib/
│   └── config/
```
## Enterprise Monorepos & Workspaces
### 11.1 Turborepo & pnpm Workspaces
In enterprise environments, sharing code across frontends and backends using a monorepo is crucial.

Setup pnpm workspace:
```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

Directory structure:
```text
├── apps/
│   ├── web/ (Next.js)
│   └── api/ (Node.js)
├── packages/
│   ├── ui/ (Shared React Components)
│   ├── types/ (Shared TypeScript Interfaces)
│   └── tsconfig/ (Shared TS configs)
```

### 11.2 Shared tsconfig
```json
// packages/tsconfig/base.json
{
  "compilerOptions": {
    "strict": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "isolatedModules": true
  }
}
```

## Modern Developer Experience (DX)
### 12.1 ESLint (Flat Config) & Prettier
```bash
npm install -D eslint prettier typescript-eslint
```

### 12.2 Husky & lint-staged
Prevent bad commits by running type-checking and linting automatically.
```bash
npx husky init
npm install -D lint-staged
```

In package.json:
```json
"lint-staged": {
  "*.ts": ["eslint --fix", "prettier --write"]
}
```

## Production Backend & NestJS
NestJS is the industry standard for enterprise TypeScript backends, providing built-in Dependency Injection and a robust module system.

### 13.1 Setup NestJS
```bash
npx @nestjs/cli new enterprise-api
```

### 13.2 Dependency Injection
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  findAll(): string[] {
    return ['User1', 'User2'];
  }
}
```

### 13.3 Controllers with Decorators
```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}
```

## CI/CD, Docker & Deployment
### 14.1 Multi-stage Dockerfile
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/index.js"]
```

### 14.2 GitHub Actions for CI
```yaml
name: CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run typecheck
      - run: npm run build
      - run: npm test
```

## TypeScript Best Practices
Strict mode always: "strict": true.

Avoid any: use unknown, then narrow with type guards or validation.

Validate all boundaries: external inputs → Zod schema → typed internal representation.

Prefer interface for public contracts, type for unions/utilities.

Use readonly and as const liberally to prevent accidental mutations.

Keep generics simple: name them meaningfully (TInput, TResponse).

Never export types you don't need – keep internal helper types local.

Use discriminated unions for state machines (loading/success/error).

Use satisfies (TS 4.9+) to validate objects without widening.

Structure by feature in monorepos, not by technical role.

Always type function return types for public functions.

Prefer esModuleInterop and ESM for modern projects.

## Real Projects
### Project 1: CLI Tool with Commander + @clack/prompts
Create a TypeScript CLI that scaffolds a project.

Requirements:

Interactive prompts (project name, template)

Creates directory with template files

Install dependencies

Key types:

```typescript
interface TemplateConfig {
  name: string;
  files: { source: string; target: string }[];
}
```
Implementation skeleton:

```typescript
#!/usr/bin/env node
import { Command } from 'commander';
import { text, select } from '@clack/prompts';
import { createProject } from './generator';

const program = new Command();
program.version('1.0.0');

program
  .command('create')
  .action(async () => {
    const projectName = await text({
      message: 'Project name:',
    });
    
    const template = await select({
      message: 'Select a template:',
      options: [
        { value: 'express-api', label: 'Express API' },
        { value: 'react-app', label: 'React App' }
      ]
    });
    
    await createProject({ projectName, template });
  });

program.parse();
```
### Project 2: REST API (Full Implementation)
As already shown in Section 5; expand with refresh tokens, file upload (multer typed), pagination helpers, and Swagger documentation via zod-to-openapi.

### Project 3: Authentication System (JWT + OAuth)
Use Passport.js with TypeScript.

Typed strategies: declare module 'passport' augment User.

Implement local strategy and Google OAuth2.

Typed req.user via module augmentation.

### Project 4: Blog Backend
CRUD for posts and comments.

Role‑based access (admin vs author).

Markdown parsing, tags, search.

Prisma relations.

### Project 5: E‑commerce Backend
Products, categories, orders, carts.

Payment integration (Stripe) with webhook typing.

Inventory management.

Type‑safe cart operations.

### Project 6: React Dashboard
Data tables with @tanstack/react-table (fully typed).

Charts (Recharts typed).

Form wizards with multi‑step validation.

State management with Zustand (typed store).

### Project 7: Next.js SaaS Application
Multi‑tenant with workspace (typed via Prisma).

Subscription billing (Stripe).

Team management.

API rate limiting with Redis.

Full‑stack type safety from DB to UI.

## Final Resources

#### TypeScript Roadmap
```text
```
Basics → Advanced Types → Generics → OOP → Tools (TSConfig, Declaration Files)
→ Testing → Backend (Node/Express) → Frontend (React/Next) → Architecture
→ Performance → Open‑source contribution.

#### TypeScript Debugging Guide
Use sourceMap: true and VS Code debugger.

debugger; statements.

tsx with --inspect.

Browser DevTools for React apps.

#### TypeScript Production Checklist
Strict mode enabled.

No any except in well‑tested wrappers.

All external data validated at runtime.

Environment variables typed and parsed at startup.

CI runs tsc --noEmit and tests.

Generated declaration files for libraries.

Path aliases configured.

Dependencies keep TypeScript types up to date.

Performance monitoring (no excessive recursive types).

All API routes have typed request/response.

## Conclusion
You now have a complete, project‑driven TypeScript mastery notebook covering everything from environment setup to production‑grade architecture. Apply the projects, experiment with the code, and you’ll be confident in any TypeScript‑based stack.