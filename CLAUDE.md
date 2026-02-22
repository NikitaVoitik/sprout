# Sprout - AI-Powered Adaptive Learning Platform

## Project Overview
Sprout is a hackathon project (single user, no auth). It builds personalised learning pathways on any topic using seven Claude-powered agents that autonomously generate 3D knowledge graphs, diagnostic assessments, and interactive tutoring.

## Monorepo Structure
```
sprout/
├── backend/     # Express 5 + TypeScript API
├── frontend/    # Next.js 16 + React
└── tracking/    # Python hand-tracking WebSocket server
```

## Commands

From the root:
```bash
npm run dev          # Start backend (port 8000) + frontend (port 3000)
npm run dev:all      # Start all 3 services including hand tracking (port 8765)
npm run install:all  # Install deps in backend/ and frontend/
npm run db:push      # Push schema directly to DB (dev shortcut)
npm run db:migrate   # Apply pending migrations
npm run db:generate  # Generate Drizzle migration files
```

From `backend/`:
```bash
npm run dev          # Start dev server with hot reload (tsx watch), port 8000
npm run build        # TypeScript compile to dist/
npm start            # Run compiled output (node dist/index.js)
```

From `frontend/`:
```bash
npm run dev          # Start Next.js dev server, port 3000
npm run build        # Build for production
npm run lint         # Run Biome linting
```

## Inspiration
Getting stuck on homework used to mean trawling through textbooks or finding a teacher on their lunch break. That struggle produced real learning. When AI arrived, any solution became one chat away — and learning suffered. Oxford University Press found 80% of students rely on AI for schoolwork and 62% say it's making them worse at learning. AI tools are here to stay; the question is how students use them. Sprout is a personal AI tutor focused on building understanding, not just giving answers.

## What it does
Describe a topic, upload notes, and watch Sprout's multi-agent system autonomously generate a 3D learning network that adapts to you. Explore your knowledge graph using natural hand movements via OpenCV hand tracking. Dive into concept nodes to learn through quiz, code, text, and drawing-based blocks. An integrated chat and voice tutor (ElevenLabs) guides learners to reach answers themselves.

## Architecture
- **Frontend** (`frontend/`): Next.js 16 + React, Three.js/React Flow for graph rendering, OpenCV hand tracking via WebSocket
- **Backend** (`backend/`): Express 5 + TypeScript, Drizzle ORM + SQLite, Anthropic SDK for Claude agents
- **Tracking** (`tracking/`): Python WebSocket server (MediaPipe + OpenCV) for hand gesture recognition
- **Single user**: Hardcoded DEFAULT_USER_ID (`00000000-0000-0000-0000-000000000000`), auto-seeded on backend startup

## Tech Stack

### Backend
- **Runtime**: Node.js with TypeScript (CommonJS, ES2022 target)
- **Framework**: Express 5
- **Database**: SQLite via better-sqlite3 + Drizzle ORM (WAL mode, foreign keys ON)
- **AI**: Anthropic SDK (`@anthropic-ai/sdk`) — all agents use Claude
- **File Storage**: AWS S3 for document uploads, pdf-parse for text extraction
- **Validation**: Zod
- **Dev**: tsx for running TS directly

### Frontend
- **Framework**: Next.js 16, React 19
- **Graph**: react-force-graph-3d, @xyflow/react, dagre, d3-force
- **UI**: Radix UI, shadcn, Tailwind CSS 4, motion
- **Linting**: Biome

## Agents (all in `backend/src/agents/`)
1. **Topic Agent** — Breaks topics into 6-10 concepts with prerequisite edges
2. **Subconcept Bootstrap Agent** — Generates 8-12 subconcepts + diagnostic questions per concept
3. **Concept Refinement Agent** — Personalises subconcept graph based on diagnostic performance
4. **Tutor Chat Agent** — Teaches subconcepts chunk-by-chunk with exercises
5. **Grade Answers Agent** — Grades diagnostic answers with scores and feedback
6. **Generate Diagnostic Agent** — Creates MCQ + open-ended diagnostic questions
7. **Review Learning Path Agent** — Post-completion enrichment and remediation

See `AGENTS.md` for full agent documentation including tools, flows, and orchestration details.

## Key patterns
- Agents use tool-calling loops (agent-loop.ts) with Claude, persisting via tools (not return values)
- Real-time SSE streaming for graph mutations (node_created, edge_created, etc.)
- DAG-based learning graphs (not linear sequences)
- Chunk-based tutoring with text/code/draw question types

## Backend Architecture

### Data Model (backend/src/db/schema.ts)
Three-level node hierarchy: **root** (topic) → **concept** → **subconcept**, stored in the `nodes` table with `type` and `parentId`. Dependencies between same-level nodes are expressed as directed edges in `nodeEdges` (forming a DAG).

Key tables: `users`, `branches`, `nodes`, `nodeEdges`, `topicDocuments`, `nodeContents`, `assessments`, `questions`, `answers`, `userNodeProgress`, `chatSessions`, `chatMessages`, `hintEvents`.

### Routes (backend/src/routes/)
- `agents.ts` — agent orchestration endpoints (topic run, concept run, node review)
- `chat.ts` — chat sessions and tutor interactions
- `nodes.ts`, `node-contents.ts`, `documents.ts` — CRUD for learning graph
- `assessments.ts`, `progress.ts` — diagnostic questions and student progress
- `users.ts`, `branches.ts` — user and branch management

All routes are mounted under `/api/` prefix. Health check at `GET /api/health`.

## Environment Variables

### Backend (`backend/.env`)
Requires: `ANTHROPIC_API_KEY`, AWS credentials for S3 (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `AWS_S3_BUCKET`). Optional: `DB_PATH` (defaults to `./sprout.db`), `PORT` (defaults to 8000).

### Frontend (`frontend/.env`)
All optional with sensible defaults: `NEXT_PUBLIC_BACKEND_ORIGIN` (default `http://localhost:8000`), `NEXT_PUBLIC_SMALL_AGENTS` (default false).

## Database Conventions
- All IDs are UUIDs (text primary key), generated with `uuid` package
- Timestamps are text columns with SQLite `datetime('now')` defaults
- JSON data stored as text columns (options, grading rubrics, response metadata) — parse with `JSON.parse()`
- Migrations live in `backend/drizzle/` directory, schema defined in `backend/src/db/schema.ts`
