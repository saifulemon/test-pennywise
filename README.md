# PennyWise

> **PennyWise** is a personal budgeting web application that helps users track income, expenses, and savings goals with visual analytics. Simple, intuitive interface designed for effective financial management without complex financial jargon.

[![Tech Stack](https://img.shields.io/badge/NestJS-E0234E?style=flat&logo=nestjs&logoColor=white)](https://nestjs.com/)
[![React](https://img.shields.io/badge/React_19-61DAFB?style=flat&logo=react&logoColor=black)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL_16-4169E1?style=flat&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![TailwindCSS](https://img.shields.io/badge/TailwindCSS_4-06B6D4?style=flat&logo=tailwindcss&logoColor=white)](https://tailwindcss.com/)

---

## Features

### For Individual Users
- 📊 **Budget Management** - Create monthly/weekly budgets per category
- 💸 **Transaction Tracking** - Log income and expenses with categories
- 🎯 **Savings Goals** - Set targets and track progress with contributions
- 📈 **Visual Analytics** - Pie charts, line graphs, spending trends
- 🚨 **Budget Alerts** - Get notified when approaching limits (80%, 100%)
- 🎨 **Custom Categories** - Add your own expense categories with icons

### For Administrators
- 👥 **User Management** - View, suspend, activate, delete user accounts
- 📂 **Category Management** - Manage system-wide expense categories
- 📊 **Platform Analytics** - User growth, transaction volumes, retention rates
- 🎫 **Support Tickets** - Handle user support requests with message threads
- ⚙️ **System Settings** - Configure default currency, session timeout, notifications

---

## Tech Stack

| Layer | Technology | Version | Port |
|-------|------------|---------|------|
| **Backend** | NestJS + TypeORM | Latest | 3100 |
| **Frontend** | React + Vite | 19 | 5273 |
| **Database** | PostgreSQL | 16 | 5532 |
| **Cache** | Redis | 7 | 6479 |
| **Deployment** | Docker Compose | - | - |

### Backend Technologies
- **Framework**: NestJS (TypeScript)
- **ORM**: TypeORM
- **Authentication**: JWT with httpOnly cookies
- **Validation**: class-validator, class-transformer
- **Documentation**: Swagger/OpenAPI
- **Password**: bcrypt (salt rounds 10+)

### Frontend Technologies
- **Framework**: React 19
- **Build Tool**: Vite
- **Styling**: TailwindCSS 4
- **State Management**: React Context / Redux (TBD)
- **HTTP Client**: Axios (with credentials support)
- **Icons**: Iconify (Lucide icon set)
- **Fonts**: Inter (sans), JetBrains Mono (mono)

---

## Project Structure

```
pennywise/
├── backend/                    # NestJS API server
│   ├── src/
│   │   ├── modules/           # Feature modules
│   │   │   ├── auth/          # Authentication (login, register, OAuth)
│   │   │   ├── budgets/       # Budget CRUD
│   │   │   ├── transactions/  # Transaction CRUD
│   │   │   ├── goals/         # Savings goals + contributions
│   │   │   ├── categories/    # Category management
│   │   │   ├── analytics/     # User analytics
│   │   │   └── admin/         # Admin-only endpoints
│   │   ├── entities/          # TypeORM entities (8 tables)
│   │   ├── dto/               # Data transfer objects
│   │   ├── guards/            # Auth & role guards
│   │   └── main.ts            # App entry point
│   └── test/                  # E2E tests
│
├── frontend/                   # React web application
│   ├── src/
│   │   ├── pages/             # 20 page components
│   │   │   ├── auth/          # Login, Signup, ForgotPassword, ResetPassword
│   │   │   ├── user/          # Dashboard, Budgets, Transactions, Goals, Analytics, Settings
│   │   │   └── admin/         # Admin Dashboard, Users, Categories, Analytics, Tickets, Settings
│   │   ├── components/        # Reusable components
│   │   ├── services/          # API service layer (axios)
│   │   ├── context/           # Auth context
│   │   ├── types/             # TypeScript types
│   │   └── router/            # Route configuration
│   └── public/                # Static assets
│
├── .claude/                    # Claude configuration & skills
├── .claude-project/            # Project documentation
│   ├── docs/                  # Detailed documentation
│   │   ├── PROJECT_KNOWLEDGE.md
│   │   ├── PROJECT_API.md
│   │   ├── PROJECT_DATABASE.md
│   │   ├── PROJECT_DESIGN_GUIDELINES.md
│   │   ├── FRONTEND_AUTH_FLOW.md
│   │   └── PROJECT_API_INTEGRATION.md
│   ├── resources/HTML/        # 20 HTML prototypes (auth, user, admin, modals)
│   └── prd/prd.pdf            # Full PRD (15 pages)
│
└── docker-compose.yml          # Service orchestration
```

---

## Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 20+ (for local development)
- Git

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

| Service | URL | Description |
|---------|-----|-------------|
| Frontend | http://localhost:5273 | React web app |
| Backend API | http://localhost:3100/api | NestJS REST API |
| Swagger Docs | http://localhost:3100/api-docs | API documentation |
| PostgreSQL | localhost:5532 | Database |
| Redis | localhost:6479 | Cache |

---

## Development

### Backend Development

```bash
cd backend

# Install dependencies
npm install

# Create .env file
cp .env.example .env

# Run migrations
npm run migration:run

# Start dev server with hot reload
npm run start:dev

# Generate new migration
npm run migration:generate -- MigrationName

# Run tests
npm run test              # Unit tests
npm run test:e2e          # E2E tests
npm run test:cov          # Coverage report
```

### Frontend Development

```bash
cd frontend

# Install dependencies
npm install

# Create .env file
cp .env.example .env

# Start Vite dev server
npm run dev

# Build for production
npm run build

# Run tests
npm run test
```

### Database Migrations

```bash
cd backend

# Generate migration from entity changes
npm run migration:generate -- AddNewFeature

# Run pending migrations
npm run migration:run

# Revert last migration
npm run migration:revert
```

---

## Environment Variables

### Backend (.env)

| Variable | Description | Required | Default | Example |
|----------|-------------|----------|---------|---------|
| `DATABASE_URL` | PostgreSQL connection string | Yes | - | `postgresql://user:pass@localhost:5532/pennywise` |
| `AUTH_JWT_SECRET` | JWT signing secret (min 32 chars) | Yes | - | `your-super-secret-key-min-32-chars` |
| `AUTH_TOKEN_COOKIE_NAME` | Access token cookie name | No | `accessToken` | `accessToken` |
| `AUTH_TOKEN_EXPIRE_TIME` | Access token expiration | No | `24h` | `24h`, `1d` |
| `AUTH_REFRESH_TOKEN_COOKIE_NAME` | Refresh token cookie name | No | `refreshToken` | `refreshToken` |
| `AUTH_REFRESH_TOKEN_EXPIRE_TIME` | Refresh token expiration | No | `7d` | `7d`, `168h` |
| `FRONTEND_URL` | Frontend URL (CORS) | Yes | `http://localhost:5173` | `https://app.example.com` |
| `MODE` | Environment mode (DEV/PROD) | Yes | `DEV` | `DEV`, `PROD` |
| `GOOGLE_CLIENT_ID` | Google OAuth client ID | No | - | `your-google-client-id` |
| `GOOGLE_CLIENT_SECRET` | Google OAuth secret | No | - | `your-google-secret` |

### Frontend (.env)

| Variable | Description | Required | Default | Example |
|----------|-------------|----------|---------|---------|
| `VITE_API_URL` | Backend API base URL | Yes | `http://localhost:3000/api` | `https://api.example.com` |

---

## API Documentation

### Authentication Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/register` | Register new user | No |
| POST | `/auth/login` | Login user (sets httpOnly cookies) | No |
| POST | `/auth/refresh` | Refresh access token | Yes |
| POST | `/auth/logout` | Logout user (clears cookies) | Yes |
| POST | `/auth/forgot-password` | Send password reset email | No |
| POST | `/auth/reset-password` | Reset password with token | No |
| GET | `/auth/google` | Google OAuth login | No |
| GET | `/auth/google/callback` | Google OAuth callback | No |

### User Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/budgets` | List user budgets | Yes |
| POST | `/budgets` | Create budget | Yes |
| GET | `/budgets/:id` | Get budget details | Yes |
| PATCH | `/budgets/:id` | Update budget | Yes |
| DELETE | `/budgets/:id` | Delete budget | Yes |
| GET | `/transactions` | List transactions (with filters) | Yes |
| POST | `/transactions` | Create transaction | Yes |
| GET | `/goals` | List savings goals | Yes |
| POST | `/goals` | Create goal | Yes |
| POST | `/goals/:id/contributions` | Add contribution | Yes |
| GET | `/categories` | List categories | Yes |
| POST | `/categories` | Create custom category | Yes |
| GET | `/analytics/spending` | Get spending breakdown | Yes |
| GET | `/analytics/trends` | Get spending trends | Yes |

### Admin Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/admin/users` | List all users | Admin |
| PATCH | `/admin/users/:id/status` | Update user status | Admin |
| DELETE | `/admin/users/:id` | Delete user | Admin |
| GET | `/admin/analytics` | Platform analytics | Admin |
| GET | `/admin/support-tickets` | List support tickets | Admin |
| PATCH | `/admin/support-tickets/:id` | Update ticket | Admin |
| GET | `/admin/categories` | Manage system categories | Admin |

**Full API documentation**: See [PROJECT_API.md](.claude-project/docs/PROJECT_API.md) or Swagger at `http://localhost:3100/api-docs`

---

## Database Schema

### Core Tables

| Table | Description | Key Columns |
|-------|-------------|-------------|
| `users` | User accounts | id (uuid), email (unique), password, display_name, currency, role, status |
| `categories` | Expense categories | id, user_id (nullable for system), name, icon, type (SYSTEM/CUSTOM) |
| `transactions` | Income/expense entries | id, user_id, category_id, type (INCOME/EXPENSE), amount, date |
| `budgets` | Spending limits | id, user_id, category_id, amount, period (WEEKLY/MONTHLY), start_date |
| `savings_goals` | Savings targets | id, user_id, name, target_amount, current_amount, target_date |
| `contributions` | Goal contributions | id, goal_id, amount, date, note |
| `support_tickets` | User support | id, user_id, subject, status, priority, messages (JSONB) |

**Full schema**: See [PROJECT_DATABASE.md](.claude-project/docs/PROJECT_DATABASE.md) with ERD

---

## Design System

### Colors
- **Primary**: Emerald #10B981 (green theme)
- **Budget Status**: Green (<80%), Yellow (80-100%), Red (>100%)
- **Semantic**: Success (emerald), Error (red), Warning (yellow), Info (blue)
- **Neutral**: Gray scale (50-900)

### Typography
- **Sans**: Inter (400, 500, 600, 700)
- **Mono**: JetBrains Mono (400, 500)

### Components
- **Buttons**: Rounded-lg, semibold, emerald primary
- **Cards**: Rounded-xl, shadow-card, p-6
- **Inputs**: Rounded-md, focus:ring-emerald
- **Icons**: Lucide via Iconify

**Full design system**: See [PROJECT_DESIGN_GUIDELINES.md](.claude-project/docs/PROJECT_DESIGN_GUIDELINES.md)

---

## Security

### Authentication Security
- ✅ **httpOnly Cookies** - Tokens inaccessible to JavaScript (XSS protection)
- ✅ **Secure Flag** - HTTPS-only in production (MITM protection)
- ✅ **SameSite Policy** - Strict in production (CSRF protection)
- ✅ **Short-lived Tokens** - 24h access tokens, 7d refresh tokens
- ✅ **CORS with Credentials** - Explicit origin allowlist
- ✅ **Password Hashing** - bcrypt with salt rounds 10+

### Best Practices
- Never store tokens in localStorage/sessionStorage
- Always use `withCredentials: true` in frontend axios
- Backend CORS must allow credentials: `credentials: true`
- Use HTTPS in production
- Rotate JWT secret regularly

**Full security documentation**: See [PROJECT_KNOWLEDGE.md](.claude-project/docs/PROJECT_KNOWLEDGE.md#security-architecture)

---

## Testing

### Backend Tests

```bash
cd backend

# Run all tests
npm run test

# E2E tests
npm run test:e2e

# Coverage report
npm run test:cov

# Watch mode
npm run test:watch
```

### Frontend Tests

```bash
cd frontend

# Run Vitest tests
npm run test

# Watch mode
npm run test:watch

# Coverage
npm run test:coverage
```

---

## Deployment

### Production Build

```bash
# Backend
cd backend
npm run build

# Frontend
cd frontend
npm run build
```

### Docker Production

```bash
# Build production images
docker-compose -f docker-compose.prod.yml build

# Start production services
docker-compose -f docker-compose.prod.yml up -d
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [CLAUDE.md](CLAUDE.md) | Quick reference for Claude Code |
| [PROJECT_KNOWLEDGE.md](.claude-project/docs/PROJECT_KNOWLEDGE.md) | Architecture, tech stack, security, environment variables |
| [PROJECT_API.md](.claude-project/docs/PROJECT_API.md) | Complete API reference (40+ endpoints) |
| [PROJECT_DATABASE.md](.claude-project/docs/PROJECT_DATABASE.md) | Database schema, ERD, migrations |
| [PROJECT_DESIGN_GUIDELINES.md](.claude-project/docs/PROJECT_DESIGN_GUIDELINES.md) | Design system (colors, typography, components) |
| [FRONTEND_AUTH_FLOW.md](.claude-project/docs/FRONTEND_AUTH_FLOW.md) | Authentication flows (login, signup, OAuth) |
| [PROJECT_API_INTEGRATION.md](.claude-project/docs/PROJECT_API_INTEGRATION.md) | Frontend-to-API mapping (20 pages) |
| [PRD](. claude-project/prd/prd.pdf) | Full Product Requirements Document |

---

## Resources

### HTML Prototypes
- **Auth Pages** (5): Splash, Login, Signup, Forgot Password, Reset Password
- **User Pages** (9): Dashboard, Budgets, Transactions, Goals, Analytics, Settings
- **Admin Pages** (6): Admin Dashboard, Users, Categories, Analytics, Tickets, Settings
- **Modals** (5): Create Budget, Add Transaction, Create Goal, Add Contribution, Delete Confirmation

Location: `.claude-project/resources/HTML/`

---

## Contributing

### Workflow

1. Create feature branch from `dev`
```bash
git checkout dev
git pull origin dev
git checkout -b feature/your-feature-name
```

2. Make changes and commit
```bash
git add .
git commit -m "feat: add your feature"
```

3. Push and create PR to `dev`
```bash
git push origin feature/your-feature-name
```

4. After review, merge to `dev`

5. `dev` → `main` for production releases

### Commit Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `style:` - Code style (formatting, no logic change)
- `refactor:` - Code refactoring
- `test:` - Add/update tests
- `chore:` - Maintenance tasks

---

## Roadmap

### Phase 1: MVP (Current)
- [x] PRD documentation
- [x] HTML prototypes (20 screens)
- [x] Database schema design
- [ ] Backend API implementation
- [ ] Frontend implementation
- [ ] Authentication & authorization
- [ ] Basic CRUD operations

### Phase 2: Enhanced Features
- [ ] Google OAuth integration
- [ ] Email notifications (budget alerts, weekly summary)
- [ ] CSV data export
- [ ] Recurring transactions
- [ ] Multi-currency support

### Phase 3: Advanced Features
- [ ] Mobile app (React Native)
- [ ] Real-time notifications (Firebase)
- [ ] AI-powered budget recommendations
- [ ] Transaction import from bank APIs
- [ ] Multi-user households

---

## Support

For questions or issues:
- Create an issue on GitHub
- Email: support@pennywise.com
- Documentation: See `.claude-project/docs/`

---

## License

MIT License - See [LICENSE](LICENSE) file for details

---

**Project Status**: 🚧 In Development

**Last Updated**: 2026-03-25

**Generated with**: [Claude Code](https://claude.com/claude-code)
