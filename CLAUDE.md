# CLAUDE.md - Animate UI

This file provides context for AI assistants working on the Animate UI codebase.

## Project Overview

Animate UI is an open-source, fully animated component distribution library built with React 19, TypeScript, Tailwind CSS 4, and Motion (formerly Framer Motion). It uses a shadcn/ui-inspired registry pattern for component distribution via CLI installation.

- **Website:** https://animate-ui.com
- **License:** MIT
- **Maintainer:** imskyleen

## Monorepo Structure

```
apps/
  www/                    # Next.js 15 documentation & demo site (Fumadocs)
packages/
  ui/                     # Core UI component library (@workspace/ui)
  eslint-config/          # Shared ESLint configs
  typescript-config/      # Shared TypeScript configs (base, nextjs, react-library)
```

### Key directories in `apps/www/`

- `registry/` — Source of truth for all distributable components, primitives, hooks, icons, and demos
- `content/docs/` — MDX documentation files (Fumadocs)
- `components/` — Site-specific demo and layout components
- `__registry__/` — Auto-generated registry index (do not edit manually)
- `public/r/` — Generated registry JSON files (do not edit manually)
- `scripts/build-registry.mts` — Script that builds the registry from source

### Key directories in `packages/ui/`

- `src/components/ui/` — Base UI components (button, input, tabs, sheet, etc.)
- `src/components/animate-ui/` — Animated component wrappers
- `src/lib/utils.ts` — `cn()` helper (clsx + tailwind-merge)
- `src/hooks/` — Custom React hooks

## Tech Stack

| Category | Technology |
|----------|-----------|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript 5.9, strict mode |
| UI | React 19 |
| Animation | Motion 12 (motion/react) |
| Styling | Tailwind CSS 4, CVA, tailwind-merge |
| Primitives | Radix UI, Base UI, Headless UI |
| Icons | Lucide React |
| Docs | Fumadocs (MDX) |
| Monorepo | Turborepo + pnpm workspaces |
| Package Manager | pnpm 10.4 |
| Node | >= 20 |

## Commands

```bash
pnpm install              # Install all dependencies
pnpm dev                  # Run dev servers (Turbo, all workspaces)
pnpm build                # Build all workspaces (Turbo)
pnpm lint                 # Lint all workspaces (Turbo)
pnpm format               # Format with Prettier
pnpm format:check         # Check formatting
pnpm registry:build       # Rebuild the component registry
```

## Git Hooks & Commit Conventions

- **Commitlint** enforces [Conventional Commits](https://www.conventionalcommits.org/):
  - `feat:` — New feature
  - `fix:` — Bug fix
  - `docs:` — Documentation only
  - `chore:` — Maintenance / tooling
  - `style:` — Formatting changes
  - `refactor:` — Code refactoring
- **Husky pre-push** runs `pnpm lint && pnpm build` — code must pass both before pushing
- **Husky commit-msg** validates commit message format

## Code Conventions

### Component Authoring

1. **Use `'use client'`** directive for any component with interactivity, state, or animations
2. **Use CVA** (class-variance-authority) for variant-based styling:
   ```tsx
   const buttonVariants = cva("base-classes", { variants: { ... } });
   ```
3. **Use `cn()`** from `@workspace/ui/lib/utils` to merge Tailwind classes
4. **Use `data-slot="component-name"`** attributes for semantic identification
5. **Use Radix `Slot` / `asChild`** pattern for polymorphic composition
6. **Use `React.ComponentProps<'element'>`** for extending HTML element props
7. **Named exports only** — no default exports for components
8. **Export both component and variants:** `export { Button, buttonVariants }`

### Animated Components

1. Wrap Radix/Base/Headless primitives with Motion for animation
2. Use `AnimatePresence` for enter/exit transitions
3. Use React Context for sharing state between compound component parts
4. Default to spring animations (`type: 'spring'`, `stiffness`, `damping`)
5. Accept a `transition` prop to allow customization

### File & Naming Conventions

- **Files:** kebab-case (`dropdown-menu.tsx`, `use-mobile.ts`)
- **Components:** PascalCase (`DropdownMenu`, `CollapsibleContent`)
- **Component parts:** Suffixed (`Trigger`, `Content`, `Provider`, `Thumb`)
- **Hooks:** `use` prefix (`useCollapsible`, `useMobile`)
- **Utilities:** lowercase functions (`cn()`, `getStrictContext()`)

### Import Paths

- `@workspace/ui/components/ui/button` — Internal UI library components
- `@workspace/ui/lib/utils` — Utility functions
- `@workspace/ui/hooks/use-mobile` — Hooks from the UI package
- `@/registry/...` — Registry components (within apps/www)
- `motion/react` — Animation library (NOT `framer-motion`)

## Registry System

### Structure

Each distributable component lives under `apps/www/registry/` in a folder with:

```
registry/primitives/[category]/[component-name]/
├── index.tsx              # Component source code
└── registry-item.json     # Metadata for CLI installation
```

### Categories

- **primitives/** — Unstyled animated components (animate, base, radix, headless, buttons, effects, texts)
- **components/** — Styled components using primitives (animate, backgrounds, base, radix, headless, buttons, community)
- **demo/** — Demo implementations for documentation
- **hooks/** — Custom React hooks
- **icons/** — Animated Lucide icon wrappers
- **lib/** — Shared utilities

### registry-item.json Format

```json
{
  "$schema": "https://ui.shadcn.com/schema/registry-item.json",
  "name": "primitives-base-switch",
  "type": "registry:ui",
  "title": "Base Switch",
  "description": "A control that indicates whether a setting is on or off.",
  "dependencies": ["motion", "@base-ui-components/react"],
  "registryDependencies": ["@animate-ui/lib-get-strict-context"],
  "files": [
    {
      "path": "registry/primitives/base/switch/index.tsx",
      "type": "registry:ui",
      "target": "components/animate-ui/primitives/base/switch.tsx"
    }
  ]
}
```

### Building the Registry

After adding/modifying registry items, always run:

```bash
pnpm registry:build
```

This processes all `registry-item.json` files, rewrites import paths for distribution, and outputs JSON files to `apps/www/public/r/`.

**Import path transformations during build:**
- `@/registry/lib/` → `@/lib/`
- `@/registry/hooks/` → `@/hooks/`
- `@/registry/` → `@/components/animate-ui/`
- `@workspace/ui/` → `@/`

## Documentation (MDX)

Docs live in `apps/www/content/docs/` and use Fumadocs with this frontmatter:

```mdx
---
title: My Component
description: Description of the component
author:
  name: Author Name
  url: https://profile-url.com
releaseDate: 2025-XX-XX
---

<ComponentPreview name="demo-my-component" />

## Installation

<ComponentInstallation name="my-component" />
```

## Adding a New Component Checklist

1. Create the component in `apps/www/registry/primitives/[category]/[name]/index.tsx`
2. Create `registry-item.json` alongside it with correct metadata
3. Create a demo in `apps/www/registry/demo/primitives/[category]/[name]/`
4. Create MDX documentation in `apps/www/content/docs/primitives/[category]/[name].mdx`
5. Run `pnpm registry:build` to update the registry
6. Verify with `pnpm build` before committing

## Formatting & Linting

- **Prettier:** trailing commas, 2-space tabs, semicolons, single quotes
- **ESLint:** TypeScript-ESLint + Prettier + Turbo rules; `--max-warnings 0` in packages/ui
- **No auto-import organization** — imports are managed manually
- **Format on save** is configured for VS Code

## Testing

There is no automated test suite. Validation relies on:
- TypeScript strict mode type checking
- ESLint linting
- `pnpm build` (Next.js build validates pages and components)
- Interactive demos in the registry serve as visual verification

## Common Pitfalls

- **Do not edit files in `__registry__/` or `public/r/`** — these are auto-generated by `pnpm registry:build`
- **Use `motion/react`** not `framer-motion` — the project uses the renamed Motion library
- **Import `cn` from `@workspace/ui/lib/utils`** not from a local utils file in apps/www
- **Always run `pnpm registry:build`** after changing any registry component or its metadata
- **Pre-push hook runs lint + build** — fix all errors before pushing
- **Node >= 20 required** — older versions will fail
