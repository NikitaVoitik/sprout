# Sprout - AI-Powered Adaptive Learning Platform

AI tutor that builds personalised 3D knowledge graphs using Claude agents. Describe a topic, upload notes, and watch a multi-agent system generate an adaptive learning network with diagnostics, tutoring, and hand-tracking navigation.

## Quick Start

### Prerequisites
- Node.js 20+
- Python 3.10 + conda (for hand tracking, optional)

### Setup

```bash
git clone https://github.com/NikitaVoitik/sprout.git
cd sprout
npm install            # installs concurrently
npm run install:all    # installs backend + frontend deps
```

### Configure Environment

```bash
cp backend/.env.example backend/.env
# Edit backend/.env — add your ANTHROPIC_API_KEY and AWS credentials
```

Frontend env is optional — defaults work for local development.

### Initialize Database

```bash
npm run db:push
```

### Start Development

```bash
npm run dev    # Starts backend (port 8000) + frontend (port 3000)
```

### Hand Tracking (Optional)

```bash
conda create -n aircanvas python=3.10 -y
conda activate aircanvas
pip install -r tracking/requirements.txt
npm run dev:all    # Starts all 3 services including tracking (port 8765)
```

## Services

| Service | Port | Description |
|---------|------|-------------|
| Backend | 8000 | Express API + Claude agents |
| Frontend | 3000 | Next.js app |
| Tracking | 8765 | Python WebSocket (hand tracking) |

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start backend + frontend |
| `npm run dev:all` | Start all 3 services |
| `npm run dev:backend` | Start backend only |
| `npm run dev:frontend` | Start frontend only |
| `npm run dev:tracking` | Start hand tracking only |
| `npm run install:all` | Install deps in both subdirs |
| `npm run build` | Build backend + frontend |
| `npm run db:push` | Push schema to database |
| `npm run db:migrate` | Run database migrations |
| `npm run db:generate` | Generate migration files |

## Architecture

See [CLAUDE.md](./CLAUDE.md) for detailed architecture documentation and [AGENTS.md](./AGENTS.md) for agent system documentation.
