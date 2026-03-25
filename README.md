# pennywise

> PennyWise is a personal budgeting web application designed for users who want to track their income, expenses, and savings goals. The app provides a simple, intuitive interface with visual analytics to help users manage their finances effectively without complex financial jargon.

## Features

- Help users create and maintain monthly budgets with ease
- Provide clear visual insights into spending patterns and trends
- Enable users to set and track progress toward savings goals
- Deliver timely alerts to prevent overspending

## Tech Stack

- **Backend**: NestJS (TypeScript, TypeORM, JWT, Swagger)
- **Frontend**: React 19 (Vite, TailwindCSS 4, shadcn/ui)
- **Database**: PostgreSQL 16
- **Cache**: Redis 7
- **Deployment**: Docker Compose

## Architecture

```
pennywise/
├── backend/              # NestJS API server
├── frontend/             # React web application
├── .claude/              # Claude configuration & skills
├── .claude-project/      # Project documentation
└── docker-compose.yml    # Service orchestration
```

## Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 20+ (for local development)

### Installation

```bash
# Clone repository with submodules
git clone --recurse-submodules https://github.com/potentialInc/pennywise.git
cd pennywise

# Start all services
docker-compose up -d

# Check service status
docker-compose ps
```

### Service URLs

- **Backend API**: http://localhost:3100
- **Frontend**: http://localhost:5273
- **PostgreSQL**: localhost:5532
- **Redis**: localhost:6479

## Development

### Backend Development

```bash
cd backend
npm install
npm run start:dev
```

### Frontend Development

```bash
cd frontend
npm install
npm run dev
```

### Database Migrations

```bash
cd backend
npm run migration:generate -- MigrationName
npm run migration:run
```

## Documentation

- **Quick Reference**: See [CLAUDE.md](CLAUDE.md) for Claude context
- **Full Documentation**: See `.claude-project/docs/`
  - [PROJECT_KNOWLEDGE.md](.claude-project/docs/PROJECT_KNOWLEDGE.md) - Architecture & goals
  - [PROJECT_API.md](.claude-project/docs/PROJECT_API.md) - API endpoints
  - [PROJECT_DATABASE.md](.claude-project/docs/PROJECT_DATABASE.md) - Database schema

## Project Structure

```
backend/
├── src/
│   ├── modules/         # Feature modules (budgets, transactions, goals)
│   ├── entities/        # TypeORM entities
│   ├── dto/             # Data transfer objects
│   └── guards/          # Auth guards
└── test/                # E2E tests
```

```
frontend/
├── src/
│   ├── components/      # Reusable components
│   ├── pages/           # Page components
│   ├── services/        # API services
│   └── types/           # TypeScript types
└── public/              # Static assets
```

## Testing

```bash
cd backend
npm run test              # Unit tests
npm run test:e2e          # E2E tests
npm run test:cov          # Coverage report
```

```bash
cd frontend
npm run test             # Vitest tests
```

## Contributing

1. Create feature branch from `dev`
2. Make changes and commit
3. Push and create PR to `dev`
4. After review, merge to `dev`
5. `dev` → `main` for production releases

## License

MIT

---

**Generated:** 2026-03-25
