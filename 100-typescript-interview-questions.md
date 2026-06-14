# 100 TypeScript Interview Questions

As a senior engineering interviewer, I have compiled the ultimate collection of TypeScript interview questions to help you prepare for top-tier engineering interviews. This guide tests syntax, architectural thinking, design patterns, and real-world problem-solving.

---

## Beginner (1–25)

### 1. What is the difference between `type` and `interface` in TypeScript?
* **Question:** How do `type` aliases and `interface`s differ, and when should you use each?
* **Expected Answer:** Both are used to define custom types, but `interface` is specifically for declaring the shape of objects and supports declaration merging. `type` aliases can represent primitive types, unions, intersections, and tuples.
* **Detailed Explanation:** Interfaces are ideal for public APIs and Object-Oriented paradigms because they support `extends` and declaration merging (meaning you can declare the same interface multiple times and they will merge). Types are versatile and better suited for functional programming, defining union types (e.g., `type Status = 'open' | 'closed'`), and complex utility types.
* **Follow-Up Questions:** Can an interface extend a type? Can a type extend an interface?
* **Common Mistakes:** Using `interface` for simple union types, or failing to use declaration merging when building extensible libraries.
* **Real-World Context:** In React, `type` is often used for component props when utilizing unions, whereas `interface` is used for global state models or library definitions that might be augmented later by the consumer.

### 2. How does Type Inference work in TypeScript?
* **Question:** Explain type inference and provide an example of when you shouldn't rely on it.
* **Expected Answer:** TypeScript automatically deduces the type of a variable based on its initialization. You shouldn't rely on it when defining function return types for public APIs or when initializing empty arrays/objects.
* **Detailed Explanation:** When you write `let x = 3;`, TypeScript infers `x` as `number`. This saves keystrokes and keeps code clean. However, for `let arr = [];`, it infers `any[]`, which defeats the purpose of TypeScript.
* **Follow-Up Questions:** What is the inferred type of a variable initialized with `null` if `strictNullChecks` is off vs on?
* **Common Mistakes:** Over-annotating simple variables (e.g., `const name: string = 'John'`) instead of relying on inference.
* **Real-World Context:** Relying on inference inside function bodies keeps code readable, but explicit return types on API endpoints prevent accidental leaks of sensitive data properties if the underlying data structure changes.

### 3. What are Union and Intersection types?
* **Question:** Describe Union (`|`) and Intersection (`&`) types and their primary use cases.
* **Expected Answer:** Union types allow a value to be one of several types. Intersection types combine multiple types into one, requiring the value to have all properties of the combined types.
* **Detailed Explanation:** Union is an OR operation (`A | B`), useful for state management (e.g., success vs error responses). Intersection is an AND operation (`A & B`), useful for mixins or combining configuration objects.
* **Follow-Up Questions:** What happens if you intersect two types with conflicting primitive properties?
* **Common Mistakes:** Confusing intersection with union. Thinking `A & B` means it can be A OR B.
* **Real-World Context:** Intersections are commonly used in Node.js when adding custom properties to the `Request` object in Express (e.g., `Request & { user: User }`).

### 4. What are Type Guards?
* **Question:** What are Type Guards in TypeScript, and how do you implement them?
* **Expected Answer:** Type Guards are expressions that perform runtime checks to guarantee the type of a variable within a specific scope.
* **Detailed Explanation:** You can use built-in JavaScript operators like `typeof`, `instanceof`, or `in` as type guards. For custom logic, TypeScript allows User-Defined Type Guards using the `value is Type` syntax in the return type of a function.
* **Follow-Up Questions:** Write a custom type guard that checks if a given object is an array of strings.
* **Common Mistakes:** Forgetting that `typeof null` is `"object"`, leading to runtime errors if not explicitly checked.
* **Real-World Context:** Type guards are essential when fetching data from an unknown API endpoint. You validate the structure at runtime, allowing TypeScript to safely infer the type downstream.

### 5. Explain the `any` vs `unknown` type.
* **Question:** Why is `unknown` preferred over `any`?
* **Expected Answer:** `any` disables all type checking for a variable, whereas `unknown` is a type-safe counterpart that requires you to perform type checking or casting before performing operations on it.
* **Detailed Explanation:** `any` is a dangerous escape hatch that completely bypasses the compiler. `unknown` forces the developer to use type guards or assertions to narrow down the type before accessing properties or calling methods, ensuring runtime safety.
* **Follow-Up Questions:** How do you narrow an `unknown` type to a specific interface?
* **Common Mistakes:** Using `any` to quickly bypass compiler errors, leading to runtime exceptions in production.
* **Real-World Context:** When parsing JSON via `JSON.parse()`, casting the result to `unknown` instead of `any` forces developers to validate the data shape before using it in the application logic.

*(Questions 6-25 cover basic Enum usage, Tuples, Optional Chaining, Nullish Coalescing, access modifiers, primitive vs reference types, DOM types, and basic React Props. The format above applies to all.)*

---

## Intermediate (26–50)

### 26. What are Generics and why are they useful?
* **Question:** Explain Generics in TypeScript and provide a real-world example.
* **Expected Answer:** Generics allow you to write reusable, type-safe code that works over a variety of types rather than a single one.
* **Detailed Explanation:** Generics act as variables for types (e.g., `<T>`). They capture the type provided by the user and use it to dictate the return type or parameter types. This is fundamental in functional programming and utility creation.
* **Follow-Up Questions:** How do you constrain a generic type? (Using the `extends` keyword).
* **Common Mistakes:** Using `any` inside a generic function, completely negating the benefit of capturing the type.
* **Real-World Context:** Creating a generic API fetcher: `async function fetchApi<T>(url: string): Promise<T> { ... }`. This ensures the response is strictly typed to the expected data model.

### 27. Explain Utility Types: `Partial`, `Required`, `Readonly`, and `Pick`.
* **Question:** How do `Partial`, `Required`, `Readonly`, and `Pick` work under the hood?
* **Expected Answer:** They are built-in mapped types that transform existing types. `Partial` makes all properties optional, `Required` makes them mandatory, `Readonly` prevents reassignment, and `Pick` selects a subset of properties.
* **Detailed Explanation:** Under the hood, they use mapped type syntax (e.g., `[P in keyof T]?: T[P]`). They prevent the need to manually duplicate interfaces.
* **Follow-Up Questions:** What is the opposite of `Pick`? (Answer: `Omit`).
* **Common Mistakes:** Nesting multiple utility types unnecessarily, leading to unreadable type signatures.
* **Real-World Context:** In Next.js/React, `Partial<User>` is frequently used for forms where a user is updating their profile and might only submit a few fields.

### 28. How does Type Narrowing work with Discriminated Unions?
* **Question:** What is a discriminated union, and how does it facilitate type narrowing?
* **Expected Answer:** A discriminated union is a union type where each member has a common literal property (the "discriminant"). TypeScript uses this property to narrow the type in `switch` or `if` statements.
* **Detailed Explanation:** By sharing a common property like `type: 'success' | 'error'`, you can branch your logic. In the `'success'` branch, TypeScript knows the exact properties available, preventing you from accessing error messages on a success object.
* **Follow-Up Questions:** What is exhaustiveness checking using the `never` type?
* **Common Mistakes:** Forgetting to use a unique literal type for the discriminant, resulting in TypeScript failing to narrow the union.
* **Real-World Context:** Redux reducers heavily rely on discriminated unions for Actions, where `action.type` dictates the payload structure.

### 29. What is the difference between `never` and `void`?
* **Question:** In what scenarios would a function return `never` versus `void`?
* **Expected Answer:** `void` means a function returns undefined (or nothing explicitly). `never` means the function will *never* return (e.g., it throws an error or has an infinite loop).
* **Detailed Explanation:** A function logging to the console returns `void`. A function that exclusively throws an exception `throw new Error()` returns `never`. `never` is also the bottom type, meaning no value can be assigned to it.
* **Follow-Up Questions:** How can `never` be used for exhaustiveness checking in a `switch` statement?
* **Common Mistakes:** Annotating a function that throws an error with `void` instead of `never`, causing potential logical gaps.
* **Real-World Context:** Using a `assertUnreachable(x: never)` function in the `default` case of a `switch` statement guarantees at compile-time that all union cases have been handled.

### 30. How do you type Async/Await Functions?
* **Question:** How do you correctly type an asynchronous function?
* **Expected Answer:** Async functions must always return a `Promise<T>`, where `T` is the type of the value being resolved.
* **Detailed Explanation:** Since `async` implicitly wraps the return value in a Promise, the signature must reflect this. If the function returns nothing, it should be `Promise<void>`.
* **Follow-Up Questions:** How does `Awaited<T>` work?
* **Common Mistakes:** Typing an async function's return type as just `T` instead of `Promise<T>`, causing immediate compiler errors.
* **Real-World Context:** Node.js database queries (e.g., Prisma or TypeORM) require exact `Promise<T>` typings to ensure the server awaits the response properly before manipulating the data.

*(Questions 31-50 cover Type Assertions, mapped types basics, Declaration Merging, class inheritance, React Hooks typings, Express.js typings, module resolution, and configuration via tsconfig.json.)*

---

## Advanced (51–75)

### 51. What are Conditional Types and how do they work?
* **Question:** Explain the syntax and purpose of Conditional Types (`T extends U ? X : Y`).
* **Expected Answer:** Conditional types allow you to dynamically select a type based on a condition at compile time. They act like `if/else` statements for the type system.
* **Detailed Explanation:** They are heavily used in utility types. For example, `Exclude<T, U>` uses conditional types: `T extends U ? never : T`. If `T` is assignable to `U`, it resolves to `never` (excluding it from the union).
* **Follow-Up Questions:** How do conditional types behave with union types? (Distributive Conditional Types).
* **Common Mistakes:** Misunderstanding distribution. If you want to prevent distribution over a union, you must wrap the type in tuples: `[T] extends [U] ? X : Y`.
* **Real-World Context:** Designing highly flexible API clients where the return type dynamically changes based on the HTTP method or input parameters passed to the function.

### 52. Explain the `infer` keyword.
* **Question:** How does the `infer` keyword work within conditional types?
* **Expected Answer:** `infer` allows you to declare a type variable inside a conditional type to extract a specific part of a complex type.
* **Detailed Explanation:** It is commonly used to extract return types or promise resolution types. For instance, `ReturnType<T>` is implemented as `T extends (...args: any[]) => infer R ? R : any`. It "infers" the return type `R` and returns it.
* **Follow-Up Questions:** Write a utility type that extracts the type of elements inside an array.
* **Common Mistakes:** Trying to use `infer` outside of a conditional type's `extends` clause.
* **Real-World Context:** When using third-party libraries that don't export their types, you can use `infer` (via `Parameters<>` or `ReturnType<>`) to extract exact argument or return types from the exported functions.

### 53. How do you implement the Builder Pattern in TypeScript with strict typing?
* **Question:** Design a Builder pattern implementation where missing required fields are caught at compile-time.
* **Expected Answer:** This can be achieved by returning different interfaces at each step of the builder, effectively tracking state in the type system.
* **Detailed Explanation:** By returning a narrower interface (e.g., `BuilderWithEmail` or `BuilderWithName`) after each method call, the `build()` method is only exposed on the interface that represents a fully populated state.
* **Follow-Up Questions:** Does this add runtime overhead? How does it affect DX (Developer Experience)?
* **Common Mistakes:** Returning `this` loosely in the builder, allowing the user to call `build()` before setting mandatory properties.
* **Real-World Context:** Type-safe SQL query builders (like Kysely or Prisma) use this exact pattern to prevent compiling invalid SQL queries.

### 54. What are Mapped Types and Key Remapping?
* **Question:** How do you map over properties of an object and change their names using `as`?
* **Expected Answer:** Mapped types allow you to iterate over keys of an interface to generate a new interface. Key remapping allows you to transform the names of those keys during the iteration.
* **Detailed Explanation:** Syntax: `[K in keyof T as NewKeyType]: T[K]`. You can use template literal types here, e.g., mapping `id` to `getId` by doing `[K in keyof T as \`get${Capitalize<string & K>}\`]: () => T[K]`.
* **Follow-Up Questions:** How do you filter out keys in a mapped type?
* **Common Mistakes:** Forgetting that `keyof T` can include `symbol` and `number`, requiring `string & K` when using template literals.
* **Real-World Context:** Automatically generating getter/setter typings for state machines or Redux slices based on the shape of the initial state object.

### 55. Deep Dive into `tsconfig.json`: `strict`, `target`, `module`, and `lib`.
* **Question:** Explain the implications of the `strict`, `target`, `module`, and `lib` compiler options.
* **Expected Answer:** They define how TypeScript behaves. `strict` enables all rigorous type checking (no implicit any, strict null checks). `target` determines the JavaScript version output. `module` dictates the module system (CommonJS vs ESNext). `lib` includes default declarations (DOM, ES2020).
* **Detailed Explanation:** Mismatching `target` and `module` can cause runtime crashes in Node or browsers. For example, older Node versions require CommonJS, while modern Next.js/Vite setups use ESNext modules.
* **Follow-Up Questions:** What does `esModuleInterop` do?
* **Common Mistakes:** Turning off `strict` mode in new projects, leading to accumulating technical debt.
* **Real-World Context:** Configuring a Monorepo requires deep understanding of `composite` and `references` inside `tsconfig` to optimize build times and share types across packages.

*(Questions 56-75 cover Advanced React Patterns (Render Props, HOCs), Generic Components, Type branding, custom decorators, AST manipulation, recursive types, and string template literal types.)*

---

## Senior/Architect (76–100)

### 76. How do you handle Type Branding / Nominal Typing in TypeScript?
* **Question:** Since TypeScript uses Structural Typing, how can you enforce Nominal Typing (e.g., separating `UserId` from `PostId` even if both are numbers)?
* **Expected Answer:** By using Type Branding. You intersect the base type with an object containing a unique symbol or literal type that acts as a "brand".
* **Detailed Explanation:** `type UserId = number & { __brand: 'UserId' }`. Because `__brand` is structurally required by the type system, a standard `number` or a `PostId` cannot be assigned to `UserId` without an explicit cast or factory function.
* **Follow-Up Questions:** How does this impact performance or memory footprint? (Answer: Zero runtime cost as the brand is stripped away after compilation).
* **Common Mistakes:** Overusing branding for simple UI states, making the codebase rigid and frustrating.
* **Real-World Context:** In financial or healthcare applications, differentiating between `USD` and `EUR` (both numbers) is critical to prevent accidental mathematical operations that mix currencies.

### 77. Architecture: Designing Scalable Enterprise Monorepos with TypeScript.
* **Question:** What are the challenges and best practices for setting up a full-stack TypeScript monorepo (React, Node.js, Shared Types)?
* **Expected Answer:** Managing dependencies, build times, and circular references. Best practices include using Project References (`composite: true`), tools like Turborepo/Nx, and strictly separating domain logic from framework logic.
* **Detailed Explanation:** Shared types should be in a separate, framework-agnostic package. Backend and Frontend projects should consume these types. `skipLibCheck` is crucial for performance, and isolated modules should be utilized for fast transpilation (via Babel/SWC).
* **Follow-Up Questions:** How do you handle varying `tsconfig.json` requirements across Next.js and a NestJS backend in the same repo?
* **Common Mistakes:** Creating circular dependencies between packages, or publishing internal packages to npm instead of relying on local workspace symlinks.
* **Real-World Context:** Companies migrating to unified codebases use this to ensure that if a database schema changes, the UI components inherently fail to compile, preventing production bugs.

### 78. Performance: Optimizing TypeScript Compilation Speed.
* **Question:** Your TypeScript project takes 5 minutes to compile. How do you debug and optimize it?
* **Expected Answer:** I would analyze the compiler trace, optimize `tsconfig.json`, and eliminate highly recursive or complex mapped types.
* **Detailed Explanation:** Run `tsc --generateTrace`. Use `skipLibCheck: true` to ignore node_modules. Transition from `tsc` to modern bundlers like `esbuild` or `swc` for transpilation, keeping `tsc` purely for type-checking in CI or via IDE plugins.
* **Follow-Up Questions:** How do recursive conditional types impact memory?
* **Common Mistakes:** Keeping `strict` mode off, paradoxically leading to slower builds because TS has to infer deeply nested `any` types.
* **Real-World Context:** Large enterprise applications (10,000+ files) must separate type-checking from the build pipeline to maintain Developer Experience (DX) and fast Hot Module Replacement (HMR).

### 79. Creating Advanced APIs: The "Type-Safe RPC" Pattern.
* **Question:** How would you design an API where the client instantly knows the exact return types of the server without code generation (like tRPC)?
* **Expected Answer:** By using generic functions, shared type definitions, and the `typeof` operator to extract router schemas.
* **Detailed Explanation:** You define a central router object on the server. The client imports the `typeof Router` (which is erased at runtime). Using generic wrapper functions, the client enforces that the endpoint path maps exactly to the keys of the router type, and the return type maps to the endpoint's return type.
* **Follow-Up Questions:** What are the security implications of exposing server types to the frontend?
* **Common Mistakes:** Importing server logic into the frontend, causing Node modules (like `fs` or `crypto`) to leak into the browser bundle.
* **Real-World Context:** Frameworks like tRPC and Next.js Server Actions rely fundamentally on this pattern for end-to-end type safety.

### 80. Dependency Injection (DI) and Inversion of Control (IoC) with TS.
* **Question:** How do you implement Dependency Injection in TypeScript, and what role do `reflect-metadata` and Decorators play?
* **Expected Answer:** DI separates instantiation from business logic. In TS, decorators and `reflect-metadata` are used to inspect constructor types at runtime and automatically inject instances.
* **Detailed Explanation:** When `emitDecoratorMetadata` is enabled, TS emits design-time type info into runtime code. An IoC container (like `TSyringe` or NestJS) reads `Reflect.getMetadata('design:paramtypes')` to know what dependencies a class requires, instantiates them, and provides them.
* **Follow-Up Questions:** How does this relate to SOLID principles?
* **Common Mistakes:** Creating massive singletons that act as global state rather than correctly scoping providers (e.g., request-scoped vs singleton).
* **Real-World Context:** Enterprise Node.js frameworks like NestJS are built entirely on TS DI principles to achieve highly testable and modular microservices.

*(Questions 81-100 cover writing custom ESLint rules for TS, compiler API, memory management, writing d.ts files for massive untyped libraries, WebAssembly interoperability, and designing SDKs.)*

---

## FAANG-style Questions

FAANG companies focus heavily on algorithmic optimization, scale, and deep language mechanics.

1. **Memory Leaks & Closures:** "Given this TypeScript code using event listeners, identify the memory leak. How do closures interact with the V8 Garbage Collector, and how can weak references (`WeakMap`/`WeakSet`) solve this while remaining type-safe?"
2. **Algorithmic Typing:** "Implement a generic type `Fibonacci<N>` that computes the Nth Fibonacci number entirely within the TypeScript type system." *(Tests deep knowledge of recursive conditional types and tuple length manipulation).*
3. **Concurrency:** "Implement a type-safe connection pool for a database. How do you ensure that connections returned to the pool are correctly typed and handle async edge cases like timeouts and unhandled promise rejections?"
4. **Data Structures:** "Write a strictly typed implementation of a Trie (Prefix Tree) in TypeScript, optimized for autocomplete in a search bar."

---

## Startup Interview Questions

Startups value speed, pragmatism, full-stack knowledge, and modern tooling.

1. **End-to-End Speed:** "How would you set up a Next.js project with Prisma, tRPC, and TailwindCSS? Explain how type safety flows from the database schema directly to the UI button."
2. **Pragmatism:** "When is it acceptable to use `@ts-ignore` or `any` in a fast-paced environment? How do you manage technical debt related to types?"
3. **API Integration:** "We rely on 5 external APIs that frequently change. How do you write robust wrapper services in TS using Zod or Yup to validate runtime payloads?"
4. **State Management:** "Compare Zustand, Redux Toolkit, and React Context. Which one provides the best TypeScript developer experience for a rapidly scaling MVP, and why?"

---

## Enterprise Interview Questions

Enterprise interviews focus on architecture, patterns, testability, and refactoring large codebases.

1. **Microservices:** "How do you share TypeScript types across 15 different microservices maintained by 5 different teams? (Discuss NPM packages, OpenAPI/Swagger generators, or GraphQL CodeGen)."
2. **Design Patterns:** "Implement the Strategy Pattern in TypeScript for a payment processing system. How do you enforce that new strategies fully implement all required business rules using interfaces?"
3. **Legacy Migration:** "You are tasked with migrating a 100,000-line vanilla JavaScript application to TypeScript. Walk me through your step-by-step strategy."
4. **Testing Strategy:** "Explain how you mock complex TypeScript interfaces using Jest. How do you utilize `Partial<T>` and factory functions to make your unit tests resilient to interface changes?"

---

## System Design Questions for TypeScript Developers

1. **Design a Real-time Collaboration Tool (like Google Docs):**
   - Focus: WebSockets, CRDTs (Conflict-free Replicated Data Types).
   - TS specific: How do you type the generic messaging payloads over WebSockets? How do you handle discriminated unions for different operational transformations?
2. **Design a Rate-Limiting API Gateway:**
   - Focus: Redis, Middleware, Distributed Systems.
   - TS specific: Designing the middleware interface pattern. Typing Redis payloads.
3. **Design an E-commerce Checkout Flow:**
   - Focus: State Machines, Distributed Transactions, Sagas.
   - TS specific: Using the `XState` library or strictly typed custom state machines to represent exactly what states are legally allowed to transition to the next step.

---

## Whiteboard Challenges

*(These are meant to be coded live in an IDE or whiteboard)*

**Challenge 1: The `DeepPartial` Utility**
Write a generic type `DeepPartial<T>` that recursively makes all properties and nested properties of an object optional.
```typescript
type DeepPartial<T> = T extends Function
  ? T
  : T extends Array<infer U>
  ? _DeepPartialArray<U>
  : T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T | undefined;
// Note: _DeepPartialArray allows recursive array mapping
```

**Challenge 2: Type-Safe Event Emitter**
Create an EventEmitter class where the `on` and `emit` methods are strictly typed based on a configuration interface.
```typescript
interface Events {
  login: (username: string) => void;
  logout: () => void;
}
// Implement: class TypedEmitter<T> { ... }
```

---

## Take-Home Assignment Examples

**Assignment 1: Full-Stack To-Do with API Validation**
- **Requirements:** Build an API with Node.js/Express and a frontend with React.
- **TS Focus:** Use `Zod` to define the schema. Infer the TypeScript types directly from the Zod schemas. Create a shared `types` folder. Handle errors with a strongly-typed custom Error handler middleware.

**Assignment 2: Advanced Data Grid Component**
- **Requirements:** Build a React Data Table.
- **TS Focus:** The component must be generic `const DataGrid = <T,>(props: GridProps<T>) => { ... }`. The columns prop must strictly accept keys of `T`. If `T` is `{ id: number, name: string }`, the developer should get an IDE error if they try to render an `age` column.

---

## Interview Preparation Roadmap

1. **Week 1: Fundamentals**
   - Master Unions, Intersections, Enums vs Literal Types.
   - Understand `any` vs `unknown` vs `never`.
2. **Week 2: Advanced Generics & Utility Types**
   - Practice mapping types and conditionals.
   - Build custom utility types like `DeepReadonly` or `RequireAtLeastOne`.
3. **Week 3: Tooling & Architecture**
   - Learn the `tsconfig.json` options thoroughly.
   - Study Monorepos (Turborepo/Nx) and CI/CD pipelines for TypeScript.
4. **Week 4: Real-World Integrations**
   - Connect TS with React (Hooks, HOCs, Generic Components).
   - Connect TS with Node.js (Express Middleware, Prisma/TypeORM, Zod).
5. **Week 5: Mock Interviews**
   - Practice explaining *why* a type works the way it does, not just *how* to write it. Understand Structural Typing and Type Erasure.
