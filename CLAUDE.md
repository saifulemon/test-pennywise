# pennywise - Claude Context

> **Quick Reference**: This file provides essential context for Claude Code interactions. For detailed documentation, see `.claude-project/docs/`.

## Project Overview

**PennyWise** is a personal budgeting web application that helps users track income, expenses, and savings goals with visual analytics.

### Goals
1. Help users create and maintain monthly budgets
2. Provide visual insights into spending patterns
3. Enable savings goal tracking
4. Deliver timely overspending alerts

### User Types
- **User (Individual)**: Manage personal finances, budgets, transactions, goals
- **Admin**: System management, user management, platform analytics, support tickets

## Quick Stack Reference

| Layer | Technology | Port |
|-------|------------|------|
| Backend | NestJS + TypeORM | 3100 |
| Frontend | React 19 + Vite + TailwindCSS | 5273 |
| Database | PostgreSQL 16 | 5532 |
| Cache | Redis 7 | 6479 |

## Key Entities

### Core Models
- **User**: Account, preferences, currency, monthly income
- **Category**: Expense categories (system + custom)
- **Transaction**: Income/expense entries
- **Budget**: Monthly/weekly spending limits per category
- **SavingsGoal**: Target amount with deadline
- **Contribution**: Goal contributions
- **SupportTicket**: User support requests

### Relationships
- User 1:N Budget, Transaction, Category, SavingsGoal
- Category 1:N Transaction, Budget
- SavingsGoal 1:N Contribution

## API Patterns

### Authentication
- **Method**: httpOnly cookie-based (XSS protection)
- **Endpoints**: `/auth/register`, `/auth/login`, `/auth/refresh`, `/auth/logout`
- **Cookies**: `accessToken` (24h), `refreshToken` (7d)

### Resource Endpoints
```
GET    /budgets          # List budgets
POST   /budgets          # Create budget
GET    /budgets/:id      # Get budget details
PATCH  /budgets/:id      # Update budget
DELETE /budgets/:id      # Delete budget
```

Similar patterns for: `/transactions`, `/goals`, `/categories`, `/analytics`

### Admin Endpoints
```
GET    /admin/users                    # List all users
PATCH  /admin/users/:id/status         # Update user status
GET    /admin/analytics                # Platform analytics
GET    /admin/support-tickets          # Support tickets
```

## Database Schema Highlights

### Key Tables
- `users`: id (uuid), email (unique), password, display_name, currency, role, status
- `categories`: id, user_id (nullable for system), name, icon, type, status
- `transactions`: id, user_id, category_id, type (INCOME/EXPENSE), amount, date, note
- `budgets`: id, user_id, category_id, amount, period, start_date, end_date, spent_amount
- `savings_goals`: id, user_id, name, target_amount, current_amount, target_date
- `contributions`: id, goal_id, amount, date, note
- `support_tickets`: id, user_id, subject, status, priority, messages (JSONB)

### Important Constraints
- Categories: UNIQUE (user_id, name) - no duplicate names per user
- Budgets: UNIQUE (user_id, category_id, period, start_date)
- System categories: user_id = NULL, type = 'SYSTEM'

## Development Commands

### Backend (NestJS)
```bash
cd backend
npm install
npm run start:dev              # Dev server with hot reload
npm run migration:generate -- MigrationName
npm run migration:run
npm run test                   # Unit tests
npm run test:e2e               # E2E tests
```

### Frontend (React)
```bash
cd frontend
npm install
npm run dev                    # Vite dev server
npm run build                  # Production build
npm run test                   # Vitest tests
```

### Docker
```bash
docker-compose up -d           # Start all services
docker-compose ps              # Service status
docker-compose logs -f backend # View logs
docker-compose down            # Stop services
```

## File Locations

### Documentation
- `.claude-project/docs/PROJECT_KNOWLEDGE.md` - Architecture, tech stack, security
- `.claude-project/docs/PROJECT_API.md` - Complete API reference (40+ endpoints)
- `.claude-project/docs/PROJECT_DATABASE.md` - Database schema, ERD (8 tables)
- `.claude-project/docs/PROJECT_DESIGN_GUIDELINES.md` - Design system (colors, typography, components)
- `.claude-project/docs/FRONTEND_AUTH_FLOW.md` - Authentication flows (login, signup, OAuth, password reset)
- `.claude-project/docs/PROJECT_API_INTEGRATION.md` - Frontend-to-API mapping (20 pages)
- `.claude-project/prd/prd.pdf` - Full PRD document (15 pages)

### Resources
- `.claude-project/resources/HTML/` - HTML prototypes:
  - **Auth**: 5 screens (splash, login, signup, forgot password, reset password)
  - **User**: 9 screens (dashboard, budgets, transactions, goals, analytics, settings)
  - **Admin**: 6 screens (dashboard, users, categories, analytics, tickets, settings)
  - **Modals**: 5 components (create budget, add transaction, create goal, add contribution, delete confirmation)
- `.claude-project/status/` - Task tracking per component
- `.claude-project/memory/` - Session memory

### Code
- `backend/src/modules/` - Feature modules (auth, budgets, transactions, goals, categories, analytics, admin)
- `backend/src/entities/` - TypeORM entities (User, Category, Transaction, Budget, SavingsGoal, Contribution, SupportTicket)
- `frontend/src/pages/` - Page components (20 screens total)
- `frontend/src/components/` - Reusable components

## Common Tasks

### Add New Feature Module
1. Create module: `nest g module modules/feature-name`
2. Create entity in `entities/feature.entity.ts`
3. Create DTO in `dto/create-feature.dto.ts`
4. Create service and controller
5. Generate migration: `npm run migration:generate -- AddFeature`
6. Run migration: `npm run migration:run`

### Add New Page
1. Create page component in `frontend/src/pages/FeatureName.tsx`
2. Add route in router configuration
3. Create API service in `frontend/src/services/feature.service.ts`
4. Add navigation link if needed

## Important Notes

### Security
- **Never** store tokens in localStorage (use httpOnly cookies)
- Always use `withCredentials: true` in frontend axios config
- CORS must allow credentials: `credentials: true`
- Password hashing: bcrypt with salt rounds 10+

### Design System (from HTML prototypes)
- **Fonts**: Inter (sans-serif, weights 400-700), JetBrains Mono (monospace, 400-500)
- **Primary Color**: Emerald #10B981 (green theme for income/safe budgets)
- **Icons**: Iconify with Lucide icon set (via CDN)
- **CSS Framework**: TailwindCSS 4 (CDN in prototypes)
- **Budget Colors**: Green (<80%), Yellow (80-100%), Red (>100%)
- **Layout**: Responsive grid, sidebar navigation, card-based design

### Conventions
- **Branches**: `main` (production), `dev` (active development)
- **Commit format**: Conventional commits (`feat:`, `fix:`, `docs:`)
- **API responses**: `{ success: boolean, data?: any, message?: string }`
- **Timestamps**: Always use UTC, store in ISO 8601 format

---

**For detailed context**, see `.claude-project/docs/`. This file is optimized for token efficiency.
