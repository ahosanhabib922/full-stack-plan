---
description: Build Project — full-stack pipeline from inspiration image + PRD to clean architecture app (Flutter, Next.js, NestJS, Express)
argument-hint: "[prd-file] [image-file]"
---

# Build Project

Full-stack build pipeline with clean architecture. Takes an inspiration image and a PRD, detects frameworks, scaffolds clean architecture layers, builds pixel-perfect UI, then wires up functionality following Presentation → Domain → Data separation throughout.

> **Architecture rule:** Before starting any phase, read `.claude/commands/clean-code-master.md` in full. Every decision made during this build — folder structure, layer separation, naming, DI, error handling, state management — must comply with the rules defined there. The clean-code-master is the single source of truth for architecture. Do not duplicate its rules here; apply them.

---

## Quick Reference

```bash
/build-project                        # Interactive — asks for PRD and image
/build-project PRD.md                 # Use PRD file, asks for image
/build-project PRD.md inspiration.png # Full auto run
```

---

## Phase 1: Input Collection

### 1.1 Locate PRD

Check `$ARGUMENTS` for a file path first. If not provided, search the project root for:

- `PRD.md`, `prd.md`, `PRD.txt`
- `README.md` with a "Product Requirements" section
- Any `.md` file containing the words "features", "requirements", or "user stories"

If not found, ask:

```text
No PRD found. Please paste your PRD content or provide the file path.
```

### 1.2 Locate Inspiration Image(s)

Check `$ARGUMENTS` for an image path first. Then search:

- `inspiration/`, `designs/`, `mockups/`, `assets/` folders
- Any `.png`, `.jpg`, `.webp` in project root

If not found, ask:

```text
No inspiration image found. Please drag an image into chat or provide the file path.
```

### 1.3 Confirm Inputs

```text
Ready to build:
  PRD:   [filename or "pasted content"]
  Image: [filename or "N/A"]
Proceeding... (type "stop" to pause)
```

---

## Phase 2: PRD Analysis

### 2.1 Stack Detection

Determine the full stack from PRD signals:

| Signal in PRD | Frontend | Backend |
| --- | --- | --- |
| "mobile app", "iOS", "Android" | Flutter | NestJS (if API needed) |
| "web app", "SaaS", "dashboard" | Next.js | Next.js API routes (or NestJS if complex) |
| "both web and mobile" | Flutter + Next.js | NestJS |
| "REST API", "microservice" | — | NestJS / Express |
| Ambiguous | Ask user | Ask user |

Ask if ambiguous:

```text
Is there a separate backend API needed?
  1. No — Next.js handles everything (API routes + UI)
  2. Yes — separate backend (NestJS or Express)
  3. Not sure — I will decide based on feature complexity
```

### 2.2 Extract Core Information

Parse and document:

- **App name** — project folder, package name, bundle ID
- **Frontend** — Flutter / Next.js / both
- **Backend** — NestJS / Express / Next.js API routes / none
- **User roles** — guest, user, admin, etc.
- **Screens / pages** — every screen mentioned
- **Features** — every feature with priority P1/P2/P3
- **Entities / data models** — User, Product, Order, etc.
- **Auth** — none / JWT / session / OAuth / phone OTP
- **Third-party integrations** — payments, maps, storage, analytics
- **Color palette / brand** — if mentioned

Print summary:

```text
PRD Analysis Complete
──────────────────────────────────────────
App Name:    [name]
Frontend:    [Flutter / Next.js]
Backend:     [NestJS / Express / None]
Screens:     [N] screens identified
Features:    [N] (P1: X, P2: Y, P3: Z)
Entities:    [User, Product, ...]
Auth:        [type]
Integrations:[list]
Architecture: Clean Architecture (Presentation → Domain → Data)
──────────────────────────────────────────
```

---

## Phase 3: Image Analysis (Vision Pass)

Analyze inspiration image(s) with vision to extract design tokens.

### 3.1 Layout Structure

- Overall layout: sidebar / top-nav / bottom-nav / full-screen / card grid
- Number of sections per screen
- Grid/column structure

### 3.2 Design Tokens

- **Colors** — primary, secondary, accent, background, surface, text (hex values)
- **Typography** — font style, sizes for h1/h2/body/caption, weights
- **Spacing** — base unit (4px or 8px grid)
- **Border radius** — sharp (0), soft (4–8px), rounded (12–16px), pill (9999px)
- **Shadows** — none / subtle / medium / heavy
- **Icons** — outline / filled / duotone style

### 3.3 Component Inventory

List every visible UI component:

- Navigation (top bar, bottom nav, sidebar, tabs)
- Buttons (primary, secondary, icon, FAB)
- Cards (type, content structure)
- Lists (simple, avatar, with actions)
- Forms (inputs, selects, toggles, date pickers)
- Media (images, video, maps)
- Modals / sheets / dialogs
- Charts / data visualizations

### 3.4 Screen Mapping

Map each PRD screen to the closest section of the inspiration image. Flag screens with no visual reference.

Print:

```text
Image Analysis Complete
──────────────────────────────────────────
Primary:      #XXXXXX
Secondary:    #XXXXXX
Background:   #XXXXXX
Surface:      #XXXXXX
Font style:   Sans-serif
Border radius: 12px
Shadow:       Subtle
Components:   [list]
──────────────────────────────────────────
```

---

## Phase 4: Project Scaffold — Clean Architecture

Read `clean-code-master.md` → Section "2.8 Feature Folder Structure" and "B.6 Backend Folder Structure" for the exact folder layout to use per detected framework.

Scaffold rules:

- Every feature gets three sub-folders: `presentation/`, `domain/`, `data/`
- `core/` holds shared errors, DI, network, theme — never feature code
- `shared/` holds reusable widgets/components with no feature dependency
- Backend: feature-first, never type-first (`controllers/`, `services/` flat folders are banned)
- Monorepo if frontend + backend are separate: `apps/` + `packages/types/` for shared types

Create all folders and placeholder files before writing any logic. Initialize with correct app name, package ID, and bundle ID from PRD.

---

## Phase 5: Core Architecture Files

Before any feature code, create the architecture foundation files.

Follow `clean-code-master.md` → Sections "2.5 Error Handling Architecture", "2.4 Dependency Injection", and "B.5 Backend Error Handling Architecture" for the exact files, class names, and patterns to use per framework.

Key files to always create:

- Error types / Failure classes — typed, not raw strings
- DI container / injection setup — all repositories and use cases registered here
- Network client — base URL, auth interceptor, error interceptor
- Global error filter / central error middleware (backend)
- Global response transformer (backend) — consistent `{ data, message, success }` shape

---

## Phase 6: Design System Setup

Apply design tokens extracted in Phase 3 before building any screen.

Flutter design system:

- `app_colors.dart` — every token as a `static const Color`
- `app_text_styles.dart` — `TextStyle` for each type scale level
- `app_theme.dart` — complete `ThemeData`: colorScheme, textTheme, elevatedButtonTheme, inputDecorationTheme, cardTheme, appBarTheme

Next.js design system:

- `tailwind.config.ts` — extend colors, fontFamily, borderRadius, boxShadow with extracted tokens
- `globals.css` — CSS variables `--color-primary`, `--radius-md`, etc.
- `shared/ui/` — base Button, Input, Card, Badge components using tokens

---

## Phase 7: Domain Layer — Build First

Build domain layer for every feature before writing any UI or data code.

Follow `clean-code-master.md` → Section "2.2 Domain Layer" for exact rules on entities, repository interfaces, and use cases per framework.

For each feature from the PRD entity list:

- Create one entity (pure class/type — no JSON, no ORM, no HTTP)
- Create one repository interface (abstract contract — no implementation)
- Create one use case per business action listed in PRD (one `call()` / `execute()` method each)
- Use case receives repository interface via constructor — never instantiates it directly

---

## Phase 8: Data Layer — Build Second

Implement data layer after domain contracts are defined.

Follow `clean-code-master.md` → Section "2.3 Data Layer" and "B.3 Repository Layer" for exact patterns per framework.

For each feature:

- Create model/DTO — serialization lives here, not in entity
- Create remote datasource — HTTP/DB calls only, returns models not entities
- Create local datasource — cache/storage only (if needed)
- Implement repository interface — coordinates datasources, maps model → entity, wraps errors
- Register everything in DI — follow Section "2.4 Dependency Injection" from clean-code-master

---

## Phase 9: Backend API (if backend detected)

Build controller/route layer after domain and data layers are complete.

Follow `clean-code-master.md` → Sections "B.1 Controller Layer", "B.2 Service Layer", "B.4 DTO and Validation", and "B.7 Full-Stack API Contract" for exact rules.

Steps:

- Create `prisma/schema.prisma` with one model per entity from PRD — run `prisma migrate dev`
- Build controllers: validate input → call use case → return response (no business logic inline)
- Wire auth guard / middleware on all protected routes
- Enforce API contract from clean-code-master: `{ success, data, message }` on all routes

---

## Phase 10: Presentation Layer — Build Last

Build UI only after domain and data layers are ready. UI connects to use cases — never to raw data sources.

### Build Order

1. Shared layout components (nav, sidebar, bottom bar, app bar)
2. Home / dashboard screen
3. Auth screens (login, register, forgot password)
4. Core feature screens — P1 priority first
5. Settings, profile, secondary screens

### Per Screen Process

For each screen:

1. Re-examine the inspiration image for this screen
2. Build layout structure (Scaffold, SafeArea, Column, Stack)
3. Add navigation component
4. Build each section top-to-bottom, left-to-right
5. Apply design tokens only — never hardcode colors or font sizes
6. Connect to controller/hook which calls use case
7. Handle all states: loading (skeleton/shimmer), empty, error, success
8. Add responsive behavior

### Pixel-Perfect Rules

- Border-radius: match exactly from vision analysis
- Spacing: follow 8px grid from vision analysis
- Colors: from design system tokens only
- Typography: from type scale tokens only
- Shadows: match weight from vision analysis
- Icons: single icon library, consistent size

### State Connection Rules

Follow `clean-code-master.md` → Sections "2.1 Presentation Layer" and "2.6 State Management Architecture" for exact rules per framework.

Core rule for all frameworks: **UI calls controller/hook → controller calls use case → use case calls repository interface. Never skip a layer.**

---

## Phase 11: Quality Pass

After build, run all three master skills automatically:

1. `/clean-code-master audit` — verify clean architecture is correct throughout
2. `/ui-review-master audit` — verify pixel-perfect against inspiration image
3. `/seo-master audit` — (Next.js only) verify SEO baseline

Fix all P1 violations before declaring complete.

---

## Phase 12: Final Report

```markdown
# Build Complete — {app-name}

**Date**: {date}
**Frontend**: {Flutter / Next.js}
**Backend**:  {NestJS / Express / None}
**Architecture**: Clean Architecture — Presentation → Domain → Data

## What Was Built

### Frontend
- {N} screens implemented
- {N} features (P1: all, P2: X of Y, P3: deferred)
- Design system: {N} tokens extracted from inspiration image
- State management: {Riverpod / React Query}

### Backend
- {N} API endpoints
- {N} use cases implemented
- Auth: {type}
- Database: {Prisma + PostgreSQL / MongoDB}

## Architecture Layers Verified

| Layer         | Flutter | Next.js | NestJS |
|---------------|---------|---------|--------|
| Presentation  | Done    | Done    | Done   |
| Domain        | Done    | Done    | Done   |
| Data          | Done    | Done    | Done   |
| DI wired      | Done    | Done    | Done   |
| Error handling| Done    | Done    | Done   |

## Screens

| Screen    | Status | Notes                        |
|-----------|--------|------------------------------|
| Home      | Done   | -                            |
| Login     | Done   | -                            |
| Dashboard | Done   | Connected to real API        |

## P2 Features (deferred)

- [ ] Feature name — reason deferred

## Next Steps

1. Add unit tests for all use cases
2. Add integration tests at repository layer
3. Replace any stub data with real API
4. Run /seo-master before launch (web)
5. Run /ui-review-master final check

## Run Commands

Frontend: {flutter run / npm run dev}
Backend:  {npm run start:dev}
```

---

## Error Handling

| Situation | Action |
|---|---|
| No PRD and no image | Ask for both before starting |
| PRD only, no image | Build with best-practice UI, no pixel-perfect pass |
| Image only, no PRD | Extract features from image, confirm before building |
| Framework ambiguous | Ask before scaffolding |
| Backend unclear | Ask if separate backend is needed |
| Feature needs unclear API | Build use case + stub datasource, flag in report |
| Third-party SDK missing | Skip integration, note in report |
