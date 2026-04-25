---
description: Clean Code Master — full-stack clean architecture audit and refactor (Flutter, Next.js, NestJS, Express)
argument-hint: "[layer|feature|backend|frontend|all]"
---

# Clean Code Master

Full-stack clean architecture audit. Enforces proper layering, separation of concerns, SOLID principles, and scalable structure across **frontend** (Flutter, Next.js) and **backend** (NestJS, Express/Node.js).

---

## Quick Reference

```bash
/clean-code-master              # Full-stack audit + auto-refactor
/clean-code-master audit        # Audit only, no changes
/clean-code-master backend      # Audit backend only (NestJS / Express)
/clean-code-master frontend     # Audit frontend only (Flutter / Next.js)
/clean-code-master domain       # Audit domain/business logic layer only
/clean-code-master data         # Audit data layer only
/clean-code-master presentation # Audit UI layer only
/clean-code-master [feature]    # Audit specific feature across all layers
```

---

## Phase 1: Detect Project & Architecture

Scan repo structure to detect what exists:

- `pubspec.yaml` → Flutter (mobile/web frontend)
- `next.config.*` → Next.js (fullstack web)
- `nest-cli.json` or `@nestjs/core` in `package.json` → NestJS (backend)
- `express` in `package.json`, no framework config → Express/Node.js (backend)
- Multiple package.json in subfolders → Monorepo (scan each)

Then detect current architecture style per project:

- Feature-first folders? (good)
- Type-first folders? (needs restructure)
- Everything flat in one folder? (needs full restructure)
- Clean Architecture layers present? (verify correctness)

Print detection summary:

```text
Detected:
  Frontend: Flutter / Next.js
  Backend:  NestJS / Express / None
  Structure: Monorepo / Single repo
  Architecture: Feature-first / Type-first / Flat
```

---

## Phase 2: Clean Architecture Layer Audit

### Target Architecture (both Flutter and Next.js)

```text
Presentation Layer  →  UI only. No business logic. No direct API calls.
Domain Layer        →  Business rules only. No framework deps. No HTTP.
Data Layer          →  API, DB, storage. No UI. No business rules.
Core / Shared       →  Utils, constants, extensions, theme, DI.
```

Strict rule: **each layer only depends on the layer below it. Never upward.**

---

### 2.1 Presentation Layer

Flutter `lib/features/[feature]/presentation/`:

- Screens / Pages: only layout and widget composition
- Controllers / ViewModels / Cubits: only UI state, delegates logic to use cases
- Widgets: purely presentational, receive data via props/params only
- No `http` / `dio` imports inside presentation
- No raw model classes from data layer used directly in UI — use ViewModels or UI models
- No `SharedPreferences` / `Hive` / `SQLite` accessed from screens

Next.js `app/` or `pages/`, `components/`:

- Page components: only layout, data fetching via hooks, no inline API calls
- Client components: UI state only (`useState`, `useEffect` for UI side effects)
- Server components: fetch data via service functions, never fetch directly inside JSX
- No `fetch()` / `axios` calls inside component body — use service layer or React Query
- No database queries in page components — use server actions or API routes

Presentation violations to flag and fix:

- API calls inside `build()` / component render
- Business logic inside screen/page files
- Direct model manipulation in UI
- `setState()` / `useState` for server data (should be React Query / Riverpod)

---

### 2.2 Domain Layer

Flutter `lib/features/[feature]/domain/`:

```text
domain/
├── entities/          # Pure Dart classes. No JSON, no fromJson, no toJson.
├── repositories/      # Abstract interfaces only. No implementation.
└── usecases/          # One class per use case. Single call() method.
```

- Entities: plain Dart classes, no Flutter imports, no `fromJson`
- Repository interfaces: abstract classes, define contract only
- Use cases: one public method (`call()`), calls repository, returns result
- No `http`, `dio`, `shared_preferences`, or any external package imported here
- No `BuildContext`, `Widget`, or any Flutter UI imported here

Next.js `lib/domain/` or `lib/features/[feature]/domain/`:

```text
domain/
├── entities/          # Pure TypeScript interfaces/classes. No fetch, no DB.
├── repositories/      # TypeScript interfaces defining contracts.
└── usecases/          # Functions or classes. Pure business logic.
```

- Entities: pure TypeScript types/interfaces, no ORM annotations
- Repository interfaces: TypeScript interfaces, no Prisma/Mongoose imports
- Use cases: pure functions receiving repository as dependency, no `fetch`, no DB

Domain violations to flag and fix:

- `fromJson` / `toJson` on domain entities (move to data layer models)
- HTTP calls inside use cases (delegate to repository)
- Database imports inside domain
- UI framework imports inside domain

---

### 2.3 Data Layer

Flutter `lib/features/[feature]/data/`:

```text
data/
├── models/            # DTOs. Has fromJson/toJson. Extends or maps to entity.
├── datasources/       # Remote (API) and Local (cache/storage) datasources.
└── repositories/      # Implements domain repository interface.
```

- Models: extend or map to domain entities. Handle JSON serialization here.
- Remote datasource: `Dio` / `http` calls only. Returns models, not entities.
- Local datasource: `SharedPreferences` / `Hive` / `SQLite` only.
- Repository implementation: coordinates remote + local datasource, maps models → entities, handles errors.
- Error handling: `try/catch` in repository, return `Either<Failure, Entity>` or throw typed exceptions.

Next.js `lib/features/[feature]/data/` or `lib/services/`:

```text
data/
├── models/            # DTOs with serialization logic.
├── datasources/       # fetch/axios wrappers, Prisma queries, Redis, etc.
└── repositories/      # Implements domain interface. Maps DTOs → entities.
```

- API routes (`app/api/`): thin controllers. Validate request → call service → return response.
- Server actions: call service/use case only. No business logic inline.
- Repository implementation: database or API access, error wrapping, DTO → entity mapping.

Data layer violations to flag and fix:

- Business logic inside repository (move to use case)
- Entity classes doing JSON parsing (move to model/DTO)
- Repository returning raw API responses instead of entities
- Missing error handling / untyped errors

---

### 2.4 Dependency Injection

Flutter DI setup:

- Configure in `lib/core/di/injection.dart` or `injection_container.dart`
- Use `get_it` or Riverpod providers
- No manual instantiation of repositories/services inside screens
- Use cases injected into controllers, not created inline

Next.js DI setup:

- Services instantiated in `lib/di/` or as singletons in `lib/services/`
- No `new Repository()` inside page components or API routes
- Use dependency injection pattern: pass repository into use case, pass use case into controller/action

DI violations to flag and fix:

- `final repo = UserRepository()` inside a screen or page
- Service/repository created with `new` inside components
- No central DI setup

---

### 2.5 Error Handling Architecture

Flutter error setup:

- Define `Failure` base class in `lib/core/errors/failures.dart`
- Define `Exception` types in `lib/core/errors/exceptions.dart`
- Repository wraps datasource exceptions → returns `Failure`
- Use case propagates failure to controller
- Controller updates UI state with error message — never crashes

Next.js error setup:

- Define error types in `lib/types/errors.ts`
- API routes return consistent error shape: `{ error: string, code: string }`
- Server actions use `try/catch`, return `{ data, error }` tuple
- Client handles both success and error cases explicitly

Error handling violations to flag and fix:

- Empty catch blocks anywhere
- `print(e)` / `console.log(e)` as only error handling
- Untyped `catch (e: any)`
- API routes returning 200 with error inside body
- Crashes propagating to UI

---

### 2.6 State Management Architecture

Flutter state rules:

- One state management solution per project (Riverpod, Bloc, or Provider — not mixed)
- State classes: immutable, use `copyWith()`
- Events (Bloc) or Notifiers (Riverpod): one per feature
- No business logic inside Bloc events/Cubit methods — delegate to use cases
- Loading / Success / Error states explicitly modeled

Next.js state rules:

- Server state: React Query / SWR — never `useState` + `useEffect` for async
- Client state: Zustand or React Context for global UI state
- Form state: React Hook Form or Formik
- No mixed patterns in same feature

State management violations to flag and fix:

- Business logic inside Bloc/Cubit directly (should call use case)
- `useState` + manual `useEffect` for API calls
- Mixed state management (Bloc + Provider in same feature)
- Mutable state objects

---

### 2.7 API Layer (Next.js specific)

Route handler pattern:

```text
Request → Validate → Call Use Case / Service → Format Response
```

- Validate request body/params before passing to service
- No business logic in route handler — delegate to service/use case
- Consistent response format across all routes
- HTTP status codes used correctly (200, 201, 400, 401, 404, 500)
- No database queries directly in route handler

Server Actions:

- In `lib/actions/` or co-located with feature
- Thin: validate → call service → return result
- Revalidate cache after mutations
- Never expose internal error messages to client

---

### 2.8 Feature Folder Structure

Every feature must follow this structure.

Flutter feature folder:

```text
lib/features/[feature]/
├── presentation/
│   ├── screens/
│   ├── widgets/
│   └── controllers/
├── domain/
│   ├── entities/
│   ├── repositories/      # abstract
│   └── usecases/
└── data/
    ├── models/
    ├── datasources/
    └── repositories/      # implementation
```

Next.js feature folder:

```text
lib/features/[feature]/
├── components/
├── hooks/
├── domain/
│   ├── entities.ts
│   ├── repository.ts      # interface
│   └── usecases.ts
└── data/
    ├── models.ts
    ├── datasource.ts
    └── repository.ts      # implementation
```

Shared across features:

```text
lib/core/          # theme, constants, utils, DI, errors, extensions
lib/shared/        # reusable widgets/components not tied to a feature
```

---

---

## Phase 2B: Backend Clean Architecture Audit

### Target Backend Architecture

```text
Controller Layer   →  HTTP in/out only. Validate request. Call service. Return response.
Service Layer      →  Business logic only. No HTTP, no DB queries directly.
Repository Layer   →  Database access only. No business logic.
Domain / Entities  →  Pure data shapes. No framework imports.
```

Strict rule: **Controller → Service → Repository. Never skip or reverse.**

---

### B.1 Controller Layer

NestJS `src/[feature]/[feature].controller.ts`:

- Decorated with `@Controller()`, `@Get()`, `@Post()` etc.
- Validates request body using DTO + `ValidationPipe`
- Calls service method — no business logic inline
- Returns formatted response — no raw DB objects
- No `prisma.*` / `typeorm.*` / `mongoose.*` imports here
- No `if/else` business decisions — delegate to service

Express `src/controllers/[feature].controller.ts`:

- Thin: parse request → call service → send response
- Input validation at controller level (Zod, Joi, or express-validator)
- No SQL/DB queries inside controller
- Consistent response shape: `{ data, message, status }`
- Error forwarded via `next(error)` — no inline try/catch logic

Controller violations to flag and fix:

- DB query directly in controller
- Business logic (`if user.role === 'admin'`) in controller
- Raw Prisma/Mongoose model returned to client (exposes internal fields)
- Missing input validation
- Inconsistent response format across routes

---

### B.2 Service Layer

NestJS `src/[feature]/[feature].service.ts`:

- Decorated with `@Injectable()`
- Injected with repository or other services via constructor
- Contains all business logic for the feature
- Throws typed exceptions (`NotFoundException`, `BadRequestException`, custom)
- No `Request` / `Response` objects imported here
- No raw SQL or ORM queries — delegates to repository

Express `src/services/[feature].service.ts`:

- Plain class or function module
- Receives repository as constructor argument (no `new Repository()` inside)
- Returns domain entities or typed DTOs — not raw DB rows
- Throws custom typed errors for controller to catch
- No `req`, `res`, `next` — pure business logic

Service violations to flag and fix:

- HTTP request object passed into service
- `res.send()` called inside service
- Direct `prisma.user.findMany()` in service (should be in repository)
- Untyped thrown errors: `throw new Error('something failed')`
- Business logic duplicated across multiple services

---

### B.3 Repository Layer

NestJS `src/[feature]/[feature].repository.ts`:

- Injected with `PrismaService` or TypeORM `EntityManager`
- Only CRUD + query methods — no business decisions
- Returns domain entities mapped from DB models, not raw Prisma types
- Every method has explicit TypeScript return type
- Handles DB-level errors only (connection errors, constraint violations)

Express `src/repositories/[feature].repository.ts`:

- Wraps Prisma / Knex / raw SQL
- No business logic — only data operations
- Maps DB rows → domain entity before returning
- Never throws HTTP errors — throws data-layer errors only

Repository violations to flag and fix:

- Business logic inside repository (`if count > limit return error`)
- Returning raw Prisma types instead of mapped entities
- HTTP error classes thrown from repository
- Multiple responsibilities in one repository (split by entity)

---

### B.4 DTO and Validation

NestJS DTOs in `src/[feature]/dto/`:

- `CreateFeatureDto`, `UpdateFeatureDto` — class-validator decorators
- Response DTOs exclude sensitive fields (passwords, internal IDs)
- `@Exclude()` on sensitive fields or use mapped types
- `PartialType`, `PickType`, `OmitType` from `@nestjs/mapped-types`

Express / generic:

- Zod schemas in `src/schemas/[feature].schema.ts`
- Validate at route entry, before passing to service
- Separate input schema from response schema

DTO violations to flag and fix:

- No validation on POST/PUT body
- Password or secret fields in response DTO
- Validation logic inside service instead of DTO
- `any` typed request body

---

### B.5 Backend Error Handling Architecture

NestJS:

- Global exception filter in `src/common/filters/http-exception.filter.ts`
- Custom exceptions extend `HttpException`
- All unhandled errors caught by global filter — no 500 leaking stack traces
- Consistent error response: `{ statusCode, message, error, timestamp }`

Express:

- Central error middleware as last `app.use()` in `app.ts`
- Custom `AppError` class with `statusCode` and `isOperational` flag
- Operational errors (user errors) vs programmer errors handled separately
- `process.on('unhandledRejection')` and `process.on('uncaughtException')` wired

Backend error violations to flag and fix:

- No global error handler — errors reaching client as raw Express errors
- Stack traces exposed in production responses
- `catch (e) { console.log(e) }` with no rethrow
- Inconsistent error response shape across routes

---

### B.6 Backend Folder Structure

NestJS (feature-first):

```text
src/
├── app.module.ts
├── main.ts
├── common/
│   ├── filters/               # Global exception filters
│   ├── guards/                # Auth guards (JWT, roles)
│   ├── interceptors/          # Logging, transform response
│   ├── decorators/            # Custom decorators
│   └── pipes/                 # Custom validation pipes
├── config/
│   ├── config.module.ts
│   └── database.config.ts
└── features/
    └── [feature]/
        ├── [feature].module.ts
        ├── [feature].controller.ts
        ├── [feature].service.ts
        ├── [feature].repository.ts
        ├── dto/
        │   ├── create-[feature].dto.ts
        │   ├── update-[feature].dto.ts
        │   └── [feature]-response.dto.ts
        └── entities/
            └── [feature].entity.ts
```

Express (feature-first):

```text
src/
├── app.ts
├── server.ts
├── common/
│   ├── middleware/
│   ├── errors/
│   └── utils/
├── config/
└── features/
    └── [feature]/
        ├── [feature].routes.ts
        ├── [feature].controller.ts
        ├── [feature].service.ts
        ├── [feature].repository.ts
        ├── [feature].schema.ts    # Zod validation
        └── [feature].types.ts
```

---

### B.7 Full-Stack API Contract

When frontend and backend are separate projects, enforce:

- API response shape consistent: `{ data: T, message: string, success: boolean }`
- Error shape consistent: `{ error: string, code: string, statusCode: number }`
- Types/interfaces shared via a `shared/` package or `types/` folder (monorepo)
- No frontend component directly using raw API response shape — map to domain model
- API versioning: `/api/v1/` prefix from day one
- HTTP status codes correct on both sides: frontend expects 201 for create, not 200

API contract violations to flag and fix:

- Frontend hardcodes `response.data.data` (double nesting)
- Backend returns different shapes from different routes
- No shared types between frontend and backend
- Frontend does `response.data?.user?.id ?? ''` because shape is unpredictable

---

### 2.9 SOLID Principles Check

#### S — Single Responsibility

- Each class/function does one thing
- Repository only handles data access
- Use case only handles one business operation
- Component only handles one UI concern

#### O — Open/Closed

- New features added via new classes, not modifying existing ones
- Repository interface allows swapping implementations (mock vs real)

#### L — Liskov Substitution

- Any implementation of a repository interface must be fully substitutable
- No implementation throwing "not implemented" exceptions

#### I — Interface Segregation

- Repository interfaces not bloated — split if one class forces implementing unrelated methods

#### D — Dependency Inversion

- High-level modules (use cases) depend on abstractions (repository interfaces)
- Low-level modules (repository implementations) inject into use cases, not the reverse

---

## Phase 3: Score

Score per layer per feature — run for both frontend and backend.

Frontend score (Flutter / Next.js):

```text
Feature: Authentication (Frontend)
┌──────────────────────────────┬────────┬──────────────────────────────────────────┐
│ Layer / Concern              │ Score  │ Issues                                   │
├──────────────────────────────┼────────┼──────────────────────────────────────────┤
│ Presentation (screens)       │  60    │ API call inside LoginScreen.build()      │
│ Presentation (state mgmt)    │  75    │ Business logic in AuthCubit              │
│ Domain (entities)            │  90    │ -                                        │
│ Domain (use cases)           │  50    │ HTTP call inside LoginUseCase            │
│ Data (models/DTOs)           │  85    │ -                                        │
│ Data (repository)            │  70    │ No error wrapping, raw exceptions thrown  │
│ Dependency Injection         │  40    │ Repository instantiated inside screen     │
│ Error Handling               │  55    │ Empty catch on line 89, no Failure types  │
│ SOLID Principles             │  65    │ UserRepository has 12 methods (too large) │
└──────────────────────────────┴────────┴──────────────────────────────────────────┘
Frontend Score: 65/100
```

Backend score (NestJS / Express):

```text
Feature: Authentication (Backend)
┌──────────────────────────────┬────────┬──────────────────────────────────────────┐
│ Layer / Concern              │ Score  │ Issues                                   │
├──────────────────────────────┼────────┼──────────────────────────────────────────┤
│ Controller layer             │  70    │ Business logic in controller             │
│ Service layer                │  55    │ Direct Prisma query in service           │
│ Repository layer             │  80    │ -                                        │
│ DTO / Validation             │  50    │ No ValidationPipe on POST body           │
│ Error handling               │  45    │ No global exception filter               │
│ Folder structure             │  75    │ Mixed feature/type folder approach       │
│ API contract                 │  60    │ Inconsistent response shape              │
│ SOLID Principles             │  70    │ -                                        │
└──────────────────────────────┴────────┴──────────────────────────────────────────┘
Backend Score: 63/100
```

---

## Phase 4: Refactor

Apply fixes in priority order. Show before/after for each change.

### P1 — Architecture violations (always fix)

- Move API calls from presentation → data layer datasource
- Move business logic from screens/cubits → use cases
- Create abstract repository interface if missing
- Wire DI — remove manual instantiation from screens
- Add Failure/Exception types if missing
- Fix empty catch blocks

### P2 — Structural fixes (fix when safe)

- Separate domain entities from data models (remove `fromJson` from entity)
- Split bloated repository interfaces
- Extract sub-widgets larger than 50 lines from screens
- Move shared widgets to `lib/shared/`
- Standardize state management pattern within feature

### P3 — Flag for manual review

- Feature folder reorganization (suggest structure, user confirms)
- State management library migration (Bloc → Riverpod, etc.)
- Breaking API contract changes

### Never

- Change business logic behavior
- Rename public API methods/types without confirmation
- Delete files without confirmation
- Modify test files

---

## Phase 5: Report

```markdown
# Clean Code Master Report — {date}

**Frontend**: {Flutter / Next.js}
**Backend**:  {NestJS / Express / None}
**Features audited**: {N}
**Overall Score**: {X}/100  (Frontend: {X} | Backend: {X})

## Frontend — Critical Fixes Applied

- [x] Moved API call from LoginScreen → AuthRemoteDataSource
- [x] Created LoginUseCase — extracted logic from AuthCubit
- [x] Created abstract AuthRepository interface in domain layer
- [x] Wired AuthRepository via get_it DI container
- [x] Added Failure types: NetworkFailure, AuthFailure
- [x] Fixed 4 empty catch blocks — now return typed Failures

## Backend — Critical Fixes Applied

- [x] Moved Prisma query from AuthService → AuthRepository
- [x] Added global ValidationPipe in main.ts
- [x] Created global HttpExceptionFilter — consistent error shape
- [x] Removed business logic from AuthController
- [x] Added response DTOs with @Exclude() on password field
- [x] Standardized all routes to return { data, message, success }

## Structural Fixes Applied

- [x] Split UserModel (data) from UserEntity (domain) — frontend
- [x] Moved shared LoadingWidget to lib/shared/widgets/ — frontend
- [x] Restructured backend from type-first to feature-first folders

## Needs Manual Review

- [ ] ProductRepository has 14 methods — consider splitting by subdomain
- [ ] Cart feature: business logic in CartCubit, no use case layer
- [ ] Shared types between frontend/backend duplicated — recommend shared package

## Architecture Score by Feature

| Feature   | Frontend | Backend | Biggest Issue                    |
|-----------|----------|---------|----------------------------------|
| Auth      | 82       | 78      | -                                |
| Dashboard | 61       | 70      | API calls in component           |
| Checkout  | 55       | 52      | No domain layer, no error filter |
| Profile   | 78       | 80      | -                                |

## Recommended Next Steps

1. Fix Checkout feature — lowest score frontend + backend
2. Create shared types package to eliminate duplication
3. Add integration tests at repository layer (backend)
4. Add unit tests for all use cases (frontend + backend services)
```

---

## Framework-Specific Notes

### Flutter Stack

- Run `dart analyze` — fix all errors before reporting
- Preferred: Riverpod + go_router + Dio + get_it + freezed
- State: `AsyncValue` (Riverpod) or sealed classes for Loading/Success/Error
- Models: use `freezed` for immutability and `copyWith`

### Next.js Stack

- Run `tsc --noEmit` — fix all type errors before reporting
- Preferred: React Query + Zod + Prisma + NextAuth
- Validate all API inputs with Zod schemas at route boundary
- Use `next-safe-action` or typed server actions for server mutations

### NestJS Backend Stack

- Run `tsc --noEmit` — fix all type errors
- Preferred: NestJS + Prisma + class-validator + Passport + @nestjs/config
- Use `ValidationPipe` globally in `main.ts`
- Use `@nestjs/mapped-types` for DTO composition
- Enable `ClassSerializerInterceptor` globally for `@Exclude()` on response DTOs
- Use `ConfigService` — never `process.env.X` directly in code

### Express Backend Stack

- Run `tsc --noEmit` — fix all type errors
- Preferred: Express + Prisma + Zod + jsonwebtoken + helmet + cors
- Validate with Zod middleware before every route handler
- Use `helmet()` and `cors()` in `app.ts`
- Central error handler as last middleware — never skip it
- `dotenv` only in `config/` — never scattered `process.env` calls

### Monorepo Full-Stack

- Use `turborepo` or `nx` for task orchestration
- Shared types package: `packages/types/` imported by both frontend and backend
- Shared validation schemas (Zod) in `packages/schemas/` — single source of truth
- API client generated from backend types (tRPC, or manual typed fetch wrapper)
- Never duplicate type definitions in frontend and backend separately
