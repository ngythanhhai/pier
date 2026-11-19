# AGENTS.md

## Skills

Skills live in `.claude/skills/` (`.agents/skills` is a symlink to it). Before starting a task that matches one of the cases below, read the corresponding `SKILL.md` first. If a task matches several cases, use every matching skill.

### Frontend / UI

| Case | Skill |
|---|---|
| Building a new UI or redesigning one (pages, components, layouts, landing pages) | `frontend-design` |
| Writing, reviewing, or refactoring React/Next.js code (components, data fetching, bundle size, re-renders) | `react-best-practices` |
| Verifying the UI actually works, debugging UI behavior, taking screenshots, reading browser logs | `webapp-testing` |
| Applying a ready-made color/font theme to a page or document | `theme-factory` |

Rules:
- For new UI, use `frontend-design` first (aesthetic direction, typography), then `react-best-practices` while writing the code.
- After changing UI, run it with `webapp-testing`. Don't rely on the code merely compiling.
- Don't use `web-artifacts-builder` for project code. It is only for HTML artifacts on claude.ai.

### Frontend architecture (FSD)

The frontend in `apps/src/` follows Feature-Sliced Design (FSD).

- Layers, top to bottom: `app` → `pages` → `features` → `entities` → `shared`. A layer may import only from layers below it, never from the same layer or above.
- Do not create a `widgets` layer. Compose page-level blocks directly in `pages`; reusable blocks go in `features` or `entities`.
- Slices expose a public API through `index.ts`. Import from the slice root, not from its internal files.
- Each slice is split into segments such as `ui`, `model`, `api`, `lib`.
- Tauri `invoke` calls live in the `api` segment of the owning slice, or in `shared/api` if they are generic.

### Claude / MCP integration

| Case | Skill |
|---|---|
| Code using the Claude API / Anthropic SDK, or questions about models, pricing, caching, tool use | `claude-api` |
| Building an MCP server (Python FastMCP or Node/TypeScript) | `mcp-builder` |

### Documents and files

| Case | Skill |
|---|---|
| Reading, creating, or editing `.docx` files | `docx` |
| Reading, creating, or editing `.xlsx`, `.csv`, `.tsv` files | `xlsx` |
| Reading, creating, or editing `.pptx` files | `pptx` |
| Working with `.pdf` files (extract text, merge, split, fill forms, OCR) | `pdf` |
| Writing structured technical docs, specs, proposals, decision docs | `doc-coauthoring` |
| Writing internal communications (status reports, incident reports, updates) | `internal-comms` |

Use these only when the file format is the main input or output. Short in-repo docs (README, `docs/*.md`) can be written directly without a skill.

### Graphic design / creative

| Case | Skill |
|---|---|
| Posters or static `.png`/`.pdf` artwork | `canvas-design` |
| Generative art with p5.js | `algorithmic-art` |
| Animated GIFs for Slack | `slack-gif-creator` |
| Applying Anthropic brand colors/fonts | `brand-guidelines` |

These are unrelated to product code. Use them only when the user asks for that exact kind of output.

### Skill management

| Case | Skill |
|---|---|
| Creating, editing, or evaluating a project skill | `skill-creator` |

## Adding a new skill

- Put it in `.claude/skills/<name>/`. Don't create a separate copy under `.agents/`.
- Add a row to the matching table above stating when to use it.
- Names in the tables are directory names. If `name` in `SKILL.md` differs from the directory name, note it (e.g. `react-best-practices` has `name: vercel-react-best-practices`).
