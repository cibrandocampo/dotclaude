---
name: frontend-patterns
description: Frontend architecture patterns and conventions for this project. Customize this skill with the project's actual stack, design tokens, and component patterns.
---

# Frontend Patterns

> **THIS SKILL HAS NOT BEEN FILLED IN.**
>
> All sections below are empty templates (`# e.g. ...`).
> **Do not assume any framework, routing, state management, styling approach, or design tokens.**
> Before writing any frontend code, ask the user:
> - What framework and build tool does this project use?
> - What is the directory structure under `frontend/`?
> - What design tokens, component patterns, and CSS conventions are established?
>
> Fill in this skill before starting any frontend work. See README.md for instructions.

## Stack

> _Fill in: framework, routing, state management, styling approach, build tool, test framework._

```
Framework:        # e.g. React 18, Vue 3, SvelteKit
Routing:          # e.g. React Router v6, Next.js App Router
State:            # e.g. Zustand, Redux Toolkit, Pinia, Context API
Styling:          # e.g. CSS Modules, Tailwind, styled-components
Build:            # e.g. Vite, webpack, Turbopack
Tests:            # e.g. Vitest + Testing Library, Jest, Cypress
```

## Design Tokens

> _Fill in: color palette, spacing scale, typography, breakpoints. These are the source of truth — never use raw values._

### Colors

```css
/* Fill in project color variables */
/* --c-primary: ;       */
/* --c-secondary: ;     */
/* --c-background: ;    */
/* --c-surface: ;       */
/* --c-text: ;          */
/* --c-text-muted: ;    */
/* --c-border: ;        */
/* --c-error: ;         */
/* --c-success: ;       */
/* --c-warning: ;       */
```

### Typography

```
/* Fill in: font family, scale, weights */
/* Base size:  */
/* Scale:      */
/* Weights:    */
```

### Spacing

```
/* Fill in: spacing scale */
/* Base unit:  */
/* Scale:      */
```

### Breakpoints

```
/* Fill in: breakpoint values */
/* sm:  */
/* md:  */
/* lg:  */
```

## Reuse-First Principle

Before writing new UI code:

1. **Check existing components** — does a component already do this?
2. **Check existing styles** — does a CSS class or token already cover this?
3. **Check existing hooks/utils** — does a hook already encapsulate this logic?

The 1/2/3 rule: once is fine, twice is a pattern, three times means extract.

## Component Patterns

> _Fill in: conventions for pages vs shared components, naming, file structure, props patterns. All frontend code lives under `frontend/` in the monorepo._

```
Root:            # frontend/
Pages:           # e.g. frontend/src/pages/, one file per route
Shared:          # e.g. frontend/src/components/, only truly reusable UI
Naming:          # e.g. PascalCase files, named exports
Co-location:     # e.g. component + styles + tests in same folder
```

## CSS Conventions

> _Fill in: how styles are written and organised._

```
Approach:        # e.g. CSS Modules, utility classes, BEM
Tokens:          # Always use design tokens, never raw values
Avoid:           # e.g. inline styles, magic numbers, global overrides
```

## API Client

> _Fill in: how the frontend calls the backend._

```
Client:          # e.g. src/api/client.js, fetch wrapper, axios instance
Auth:            # e.g. JWT in header, cookie, session
Error handling:  # e.g. throws on non-2xx, global error boundary
```

## State Management

> _Fill in: what lives where._

```
Server state:    # e.g. React Query, SWR, manual fetch + useState
UI state:        # e.g. local useState, Zustand store
Global state:    # e.g. auth context, theme context
```

## Testing

> _Fill in: what to test and how._

```
Unit:            # e.g. Vitest + Testing Library, test behaviour not implementation
E2E:             # e.g. Playwright, cover critical user flows
Mocking:         # e.g. MSW for API mocks
```
