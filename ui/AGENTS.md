<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

## Nokify frontend conventions

Follow the [project rules](../AGENTS.md). Preserve the Next.js-managed block above
and consult the installed framework documentation before application changes.

### Structure and dependencies

Keep the existing root-level `app/`; do not introduce a parallel `src/` tree.
Create these folders only when actual code needs them:

```text
app/                         Routes, layouts, and page composition
features/<feature-name>/     Feature-specific components, hooks, and logic
shared/
  components/
    buttons/                 Reusable buttons
    icons/                   Reusable icon components
    forms/                   Reusable form controls
    feedback/                Loading, error, and status components
  hooks/                     Reusable React hooks
  helpers/                   Pure reusable utility functions
  types/                     Types shared across features
```

- Pages compose features; features may import shared modules; shared modules must
  not import features or routes. Avoid circular and unnecessary cross-feature
  dependencies. Keep feature-only code inside its feature.
- Use the existing `@/*` alias, such as `@/shared/components/buttons/Button`.
  Relative imports within a small module are fine. Do not duplicate aliases.
- Shared UI components must not contain feature-specific API calls or business
  rules. Keep state near consumers; add global state only for actual shared needs.
- Keep server-only dependencies and secrets out of client module imports. Use
  framework server-only guards for sensitive server modules. Add client boundaries
  only where interaction or browser APIs require them.

### Naming and readability

- Use `kebab-case` folders (`notification-settings`, `buttons`); `PascalCase`
  component files/exports (`Button.tsx`, `Button`) and types (`ButtonProps`).
- Use `camelCase` utility files/functions (`formatDate.ts`, `formatDate`) and
  `use`-prefixed hooks/files (`useNotifications.ts`, `useNotifications`).
- Use descriptive `camelCase` variables, `is`/`has`/`can`/`should` booleans, and
  `UPPER_SNAKE_CASE` constants. Preserve framework-required names such as
  `page.tsx`, `layout.tsx`, and `route.ts`.
- Preserve strict TypeScript checking. Prefer concrete types or narrowed `unknown`
  over `any`; justify unavoidable exceptions locally. Validate external data at
  runtime when its shape or trust boundary requires it.
- Use two-space indentation and readable multiline JSX consistent with existing
  code. Keep rendering, data access, and business transformations focused.

### Accessible and secure UI

- Use semantic controls, associated labels, keyboard access, visible focus, and
  accessible names for icon-only buttons. Hide decorative icons from assistive
  technology. Do not convey status through color alone.
- Reusable buttons support disabled/loading states and intentional submission;
  default a generic button's type to `button`.
- Provide truthful loading, empty, error, and success states. Wait for confirmed
  server success and do not silently discard failed requests.
- Client permission checks are presentation only; Laravel must enforce access.
  Next.js server endpoints/actions, if introduced, must also enforce their own
  validation and authorization boundaries.
- Put only public configuration in `NEXT_PUBLIC_*`. Keep secrets out of browser
  code and do not persist authentication secrets in browser storage by default.
  Choose authentication architecture before implementing protected features.
- Prefer React text rendering. Raw HTML requires a reviewed sanitization strategy;
  never directly render untrusted HTML.

### Checks

- From `ui/`, run `npm run lint`, `npx tsc --noEmit`, and `npm run build` for
  relevant application changes using installed dependencies. Run `npm audit` for
  dependency changes and security reviews.
- Verify interactive changes in a real browser, including keyboard use and
  loading/error states. Add meaningful behavior tests when a suitable runner
  exists; linting alone does not verify interactions.
- No frontend test runner is currently configured in package scripts. Adding one
  is a separate tooling change, not implied by this documentation.
- Report checks that could not run and their causes. Documentation-only changes
  require link, consistency, and diff checks rather than application builds.
