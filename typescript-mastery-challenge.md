# TypeScript Mastery Challenge

## Project Overview
**Project**: "Build a Real-Time Collaborative Platform"

This project serves as a practical TypeScript certification exam designed to accurately evaluate whether a developer truly understands TypeScript. By building a real-time collaborative platform (e.g., Collaborative Whiteboard, Collaborative Notes, or Collaborative Kanban), candidates will demonstrate their ability to architect, implement, and maintain complex, type-safe applications. 

The project is specifically calibrated to be difficult enough to clearly distinguish between **Beginner**, **Intermediate**, **Advanced**, and **Senior** TypeScript Engineers.

## Requirements
- **Real-Time Collaboration**: Multiple users must be able to view and edit the same document or board simultaneously.
- **State Synchronization**: Changes made by one user must be reflected across all connected clients with minimal latency.
- **Conflict Resolution**: Implement a mechanism to handle conflicting concurrent edits (e.g., Operational Transformation (OT) or Conflict-free Replicated Data Types (CRDTs)).
- **User Presence**: Display active users, their cursors, or their current selection in real-time.
- **Persistence**: Save the state of the collaboration session to a database and allow users to resume sessions later.
- **Offline Support**: Basic offline capabilities with state reconciliation upon reconnection.

## Technical Constraints
- The entire project (frontend and backend) must be written in strict TypeScript (`"strict": true` in `tsconfig.json`).
- No `any` types allowed under any circumstances. `unknown` should be used where necessary and properly narrowed.
- Avoid using `@ts-ignore` or `@ts-expect-error` unless absolutely necessary, and only if accompanied by a detailed explanatory comment.
- Ensure zero type errors during the build process.

## Architecture Expectations
- **Monorepo Structure**: Utilize a monorepo setup (e.g., npm/yarn/pnpm workspaces, Turborepo, or Nx) to effectively share types between the frontend and backend.
- **Separation of Concerns**: Maintain clear boundaries between the network layer, state management, and UI components.
- **Scalability**: The architecture should logically support scaling to multiple collaboration rooms and many concurrent users.

## TypeScript Requirements
The candidate must actively demonstrate proficiency in the following TypeScript features within the project:

* **Type Narrowing**: Properly narrowing `unknown` types from API payloads, WebSockets, or third-party data.
* **Generics**: Creating reusable, type-safe data structures, API wrappers, and React/Vue components.
* **Utility Types**: Effectively utilizing built-in utility types like `Partial`, `Pick`, `Omit`, `Record`, `ReturnType`, `Awaited`, etc.
* **Advanced Types**: Implementing intersection types, union types, and branded/opaque types for safer domain modeling.
* **Type Guards**: Writing custom type guard functions (`x is Type`) to validate complex data structures at runtime.
* **Conditional Types**: Creating dynamic types that depend on input type parameters for flexible API definitions.
* **Mapped Types**: Transforming existing types into new configurations (e.g., making all properties of a complex object readonly, optional, or observable).
* **Module Augmentation**: Extending external library types to add missing properties or strongly type environment variables.
* **Declaration Files**: Writing `.d.ts` files for any untyped third-party modules or global window variables used in the project.

## Backend Requirements
- **Technology Stack**: Node.js with a framework like Express, Fastify, or NestJS.
- **Real-Time Communication**: WebSockets (e.g., `ws` or `Socket.io`) for real-time bidirectional communication.
- **Strongly Typed APIs**: Implement strictly typed request/response payloads (e.g., using tRPC, GraphQL, or REST combined with Zod/Yup validation).
- **Database**: Use an ORM or Query Builder with strong TypeScript support (e.g., Prisma, Drizzle ORM, or Kysely).

## Frontend Requirements
- **Technology Stack**: React, Vue, or Angular.
- **State Management**: Robust, strictly typed state management for both local UI state and shared collaborative state.
- **Type-Safe Components**: Strongly typed props, component state, and event handlers.
- **Error Handling**: Graceful error handling using typed error boundaries and typed API error responses.

## Testing Requirements
- **Unit Tests**: Test core business logic, utility functions, and complex type transformations.
- **Type Testing**: Include specific tests to verify that type constraints and generics behave as expected (e.g., using `tsd` or `expect-type`).
- **Integration Tests**: Test WebSocket event handling and API endpoints with proper mocked types.

## Performance Requirements
- **Optimized Rendering**: Prevent unnecessary re-renders when remote cursor positions or deep object trees update.
- **Efficient Payload Sizes**: Send minimal, optimized payloads over WebSockets.
- **Debouncing/Throttling**: Properly type and implement rate-limiting for high-frequency events (like mouse movements or keystrokes).

## Security Requirements
- **Input Validation**: Strongly typed runtime validation (e.g., Zod, Valibot) for all incoming WebSocket messages and HTTP requests.
- **Authentication/Authorization**: Secure user sessions and strictly type access control (ensuring users can only access permitted collaboration rooms).
- **XSS Prevention**: Safe rendering of user-generated content in the collaborative workspace.

## Evaluation Rubric
### Scoring System (0 - 100 Points)

* **Type Safety & Strictness (25 Points)**: Zero `any` types, proper use of `unknown`, strict compiler options enabled.
* **Advanced TypeScript Usage (25 Points)**: Meaningful, practical application of Generics, Conditional/Mapped Types, and custom Type Guards.
* **Architecture & Code Organization (20 Points)**: Clean monorepo structure, effective code sharing across boundaries, separation of concerns.
* **Feature Completeness (15 Points)**: Real-time sync, conflict resolution, presence, and persistence all functioning as required.
* **Testing & Robustness (15 Points)**: Solid test coverage, type-level testing, and impenetrable runtime validation.

### Developer Classification

* **Beginner (0 - 40)**: Relies heavily on `any` or very basic types. Struggles with Generics. State synchronization is buggy. Types are mostly used as afterthoughts rather than design tools.
* **Intermediate (41 - 70)**: Avoids `any`, uses interfaces and basic generics effectively. Real-time features work but may lack proper conflict resolution. Uses `as` casting too frequently instead of proper narrowing.
* **Advanced (71 - 90)**: Uses complex mapped/conditional types correctly. Shares types across the monorepo seamlessly. Implements robust runtime validation and functional conflict resolution. Little to no type casting.
* **Senior TypeScript Engineer (91 - 100)**: Masterful use of the type system to make invalid states unrepresentable. Creates elegant, reusable generic abstractions. Implements complex patterns (like CRDTs) flawlessly with perfect end-to-end type safety. Includes comprehensive type-level tests.

## Common Mistakes
- **Type Casting (`as`)**: Overusing the `as` keyword to silence the compiler instead of using proper type narrowing or runtime validation.
- **Leaky Abstractions**: Failing to share types properly between frontend and backend, leading to manually duplicated or mismatched types.
- **Ignoring Edge Cases**: Not typing or handling potential `null`, `undefined`, or specific error states correctly.
- **Overcomplicating Types**: Creating "type gymnastics" that are impossible to read or maintain when a simpler interface would suffice.
- **Assuming Network Trust**: Relying solely on compile-time types for WebSocket payloads without validating the incoming data at runtime.

## Senior-Level Expectations
A candidate performing at a Senior level is expected to:
- Design types that actively guide the developer experience (DX), providing excellent autocomplete and early error detection in the IDE.
- Use branded types or opaque types to enforce domain logic at compile-time (e.g., preventing a `UserId` from being accidentally passed as a `RoomId`).
- Seamlessly integrate runtime validation with static types to guarantee data integrity across network boundaries.
- Write code that is self-documenting through its expressive type signatures.
- Understand the performance implications of complex types on the TypeScript compiler and IDE language server.

## Gold Standard Solution Characteristics
- **End-to-End Type Safety**: A single change to a database model immediately flags compilation errors in backend handlers, API routers, and frontend components if not updated.
- **State-Machine Driven**: Complex UI and interaction states are modeled as strictly typed finite state machines, ensuring impossible states cannot be reached.
- **Immaculate Error Handling**: Every possible failure mode is typed, caught, and handled gracefully on both the client and server.
- **No Type Gymnastics**: The codebase achieves high type safety without resorting to excessively cryptic, unreadable mapped or conditional type puzzles.