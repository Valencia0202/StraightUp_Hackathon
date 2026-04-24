# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)
- **AI**: OpenAI via Replit AI Integrations (gpt-5.4, streaming chat)

## Artifacts

- **chatbot** (react-vite, `/`): AI chatbot web app with streaming conversations (Editorial design — warm cream paper, Cormorant Garamond serif, dark library sidebar, sage green accent). Operates in **tutor mode** (free-form streaming text). Sidebar filters to mode=tutor conversations only.
- **agency-workspace** (react-vite, `/agency-workspace/`): Standalone duplicate of the chatbot UI now running in enforced **Agency Mode**. The first AI turn always returns a structured `{type:"choice", question, options[3]}` (rendered as 3 buttons), and every subsequent turn returns `{type:"answer", steps[]}` (rendered as numbered steps). **Calc sub-flow**: when the user message is detected as arithmetic (server-side `isCalculationRequest` — `+`/`*`/`/`/`×`/`÷` adjacent to digits, `-`/`x` requiring whitespace boundaries, short bare-math expressions ≤30 chars made only of digits/operators/parens, calc verbs like "calculate"/"solve"/"compute" with a digit, or "what is …" with word/symbolic operators), the server returns `{type:"steps", message, steps[], finalAnswer:null}` first (a "Show Final Answer" button is rendered). Only when the user clicks that button does the FE send the sentinel `"Please reveal the final answer now."`, which the server treats as the reveal request and returns `{type:"final", answer, explanation}`. The server bans answer leaks in steps with `ANSWER_REVEAL_PATTERNS` (rejects `=\s*-?\d`, "the answer is", "final answer", "equals N", "thus/therefore/so/we get N"); on validation failure it retries up to 3× then persists a safe structured fallback. The frontend is hard-enforced to never display raw text — anything that fails JSON parse renders as a "malformed" bubble. Sidebar filters to mode=agency conversations only.
- **api-server** (Express 5, `/api`): REST API backend
- **mockup-sandbox** (design canvas): component preview server

## Features

- Multi-conversation chat interface
- AI-powered responses using gpt-5.4 with streaming (SSE)
- Persistent conversations and message history via PostgreSQL
- Clean sidebar for managing conversations (active highlight, inline rename via pencil icon — Enter saves, Esc cancels)
- Dynamic conversation titles: new sessions start as "New Conversation" and are auto-renamed (≤5 words derived from the first user message) on the first send. Auto-rename runs at most once per conversation, gated by an atomic conditional UPDATE (title still in placeholder allowlist AND user-message count = 1).
- Manual rename via PATCH `/api/openai/conversations/:id`
- Calm sage green theme

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally

## Key Files

- `lib/api-spec/openapi.yaml` — OpenAPI contract
- `lib/db/src/schema/conversations.ts` — conversations table
- `lib/db/src/schema/messages.ts` — messages table
- `artifacts/api-server/src/routes/openai/index.ts` — chat API routes
- `artifacts/chatbot/src/` — React frontend

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
