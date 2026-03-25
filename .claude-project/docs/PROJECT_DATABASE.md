# Database Schema: pennywise

## Overview

- **Database**: PostgreSQL
- **ORM**: TypeORM / Prisma
- **Migrations**: [Location of migration files]

## Entity Relationship Diagram

```
┌──────────────────┐
│      User        │
├──────────────────┤
│ PK id            │
│    email         │
│    password      │
│    displayName   │
│    currency      │
│    monthlyIncome │
│    role          │
│    status        │
└──────────────────┘
         │
         │ 1:N
         │
         ├────────────────────────────────────────────────────┐
         │                                                    │
         ▼                                                    ▼
┌──────────────────┐                              ┌──────────────────┐
│    Category      │                              │     Budget       │
├──────────────────┤                              ├──────────────────┤
│ PK id            │◄──┐                          │ PK id            │
│ FK userId        │   │                          │ FK userId        │
│    name          │   │                          │ FK categoryId    │──┐
│    icon          │   │                          │    amount        │  │
│    type          │   │                          │    period        │  │
│    status        │   │                          │    startDate     │  │
│    usageCount    │   │                          │    endDate       │  │
└──────────────────┘   │                          └──────────────────┘  │
         │             │                                   │            │
         │ 1:N         │ N:1                               │ 1:N        │
         │             │                                   │            │
         ▼             │                                   ▼            │
┌──────────────────┐   │                          ┌──────────────────┐ │
│   Transaction    │───┘                          │  BudgetAlert     │ │
├──────────────────┤                              ├──────────────────┤ │
│ PK id            │                              │ PK id            │ │
│ FK userId        │                              │ FK budgetId      │─┘
│ FK categoryId    │                              │    threshold     │
│    type          │                              │    triggered     │
│    amount        │                              │    sentAt        │
│    date          │                              └──────────────────┘
│    description   │
│    note          │                    ┌──────────────────┐
└──────────────────┘                    │   SavingsGoal    │
         │                              ├──────────────────┤
         │                              │ PK id            │
         │ N:1                          │ FK userId        │
         │                              │    name          │
         │                              │    targetAmount  │
         │                              │    currentAmount │
         │                              │    targetDate    │
         │                              │    monthlyTarget │
         │                              └──────────────────┘
         │                                       │
         │                                       │ 1:N
         │                                       │
         │                                       ▼
         │                              ┌──────────────────┐
         │                              │  Contribution    │
         │                              ├──────────────────┤
         │                              │ PK id            │
         │                              │ FK goalId        │
         │                              │    amount        │
         │                              │    date          │
         │                              │    note          │
         │                              └──────────────────┘
         │
         │
         ▼
┌──────────────────┐
│ SupportTicket    │
├──────────────────┤
│ PK id            │
│ FK userId        │
│    subject       │
│    status        │
│    priority      │
│    messages      │
└──────────────────┘
```

## Entity Relationships

### One-to-Many (1:N)
- User → Budget (one user has many budgets)
- User → Transaction (one user has many transactions)
- User → Category (one user has many custom categories)
- User → SavingsGoal (one user has many savings goals)
- User → SupportTicket (one user has many support tickets)
- Category → Transaction (one category has many transactions)
- Budget → BudgetAlert (one budget has many alerts)
- SavingsGoal → Contribution (one goal has many contributions)

### Many-to-One (N:1)
- Transaction → Category (many transactions belong to one category)
- Budget → Category (many budgets belong to one category)

## Tables

### users

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| email | VARCHAR(255) | No | - | Unique email |
| password | VARCHAR(255) | No | - | Hashed password (bcrypt) |
| display_name | VARCHAR(100) | No | - | User's display name |
| currency | VARCHAR(3) | No | 'USD' | Preferred currency (ISO 4217) |
| monthly_income | DECIMAL(12,2) | Yes | NULL | Monthly income for recommendations |
| role | ENUM('USER', 'ADMIN') | No | 'USER' | User role |
| status | ENUM('ACTIVE', 'SUSPENDED') | No | 'ACTIVE' | Account status |
| created_at | TIMESTAMP | No | now() | Account creation time |
| updated_at | TIMESTAMP | No | now() | Last update time |
| last_login | TIMESTAMP | Yes | NULL | Last login timestamp |

**Constraints:**
- UNIQUE (email)

### categories

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| user_id | uuid | Yes | NULL | FK to users (NULL = system category) |
| name | VARCHAR(100) | No | - | Category name |
| icon | VARCHAR(50) | No | 'receipt' | Icon identifier |
| type | ENUM('SYSTEM', 'CUSTOM') | No | 'CUSTOM' | Category type |
| status | ENUM('ACTIVE', 'INACTIVE') | No | 'ACTIVE' | Category status |
| usage_count | INTEGER | No | 0 | Number of transactions using this |
| created_at | TIMESTAMP | No | now() | Creation time |
| updated_at | TIMESTAMP | No | now() | Last update |

**Constraints:**
- FOREIGN KEY user_id REFERENCES users(id) ON DELETE CASCADE
- UNIQUE (user_id, name) - prevents duplicate category names per user

**Note:** System categories have `user_id = NULL` and `type = 'SYSTEM'` (e.g., Food, Transport, Entertainment)

### transactions

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| user_id | uuid | No | - | FK to users |
| category_id | uuid | No | - | FK to categories |
| type | ENUM('INCOME', 'EXPENSE') | No | 'EXPENSE' | Transaction type |
| amount | DECIMAL(12,2) | No | - | Transaction amount |
| date | DATE | No | CURRENT_DATE | Transaction date |
| description | VARCHAR(255) | Yes | NULL | Auto-suggested description |
| note | TEXT | Yes | NULL | User's optional note |
| created_at | TIMESTAMP | No | now() | Creation time |
| updated_at | TIMESTAMP | No | now() | Last update |

**Constraints:**
- FOREIGN KEY user_id REFERENCES users(id) ON DELETE CASCADE
- FOREIGN KEY category_id REFERENCES categories(id) ON DELETE RESTRICT
- CHECK (amount > 0)

### budgets

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| user_id | uuid | No | - | FK to users |
| category_id | uuid | No | - | FK to categories |
| amount | DECIMAL(12,2) | No | - | Budget limit |
| period | ENUM('WEEKLY', 'MONTHLY') | No | 'MONTHLY' | Budget period |
| start_date | DATE | No | - | Period start date |
| end_date | DATE | No | - | Period end date |
| spent_amount | DECIMAL(12,2) | No | 0 | Current spent amount (calculated) |
| created_at | TIMESTAMP | No | now() | Creation time |
| updated_at | TIMESTAMP | No | now() | Last update |

**Constraints:**
- FOREIGN KEY user_id REFERENCES users(id) ON DELETE CASCADE
- FOREIGN KEY category_id REFERENCES categories(id) ON DELETE RESTRICT
- UNIQUE (user_id, category_id, period, start_date)
- CHECK (amount > 0)
- CHECK (end_date > start_date)

### budget_alerts

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| budget_id | uuid | No | - | FK to budgets |
| threshold | INTEGER | No | 80 | Alert threshold percentage |
| triggered | BOOLEAN | No | false | Whether alert has been triggered |
| sent_at | TIMESTAMP | Yes | NULL | When alert was sent |
| created_at | TIMESTAMP | No | now() | Creation time |

**Constraints:**
- FOREIGN KEY budget_id REFERENCES budgets(id) ON DELETE CASCADE
- CHECK (threshold > 0 AND threshold <= 100)

### savings_goals

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| user_id | uuid | No | - | FK to users |
| name | VARCHAR(100) | No | - | Goal name |
| target_amount | DECIMAL(12,2) | No | - | Target amount to save |
| current_amount | DECIMAL(12,2) | No | 0 | Current saved amount |
| target_date | DATE | No | - | Target completion date |
| monthly_target | DECIMAL(12,2) | No | - | Required monthly savings |
| created_at | TIMESTAMP | No | now() | Creation time |
| updated_at | TIMESTAMP | No | now() | Last update |

**Constraints:**
- FOREIGN KEY user_id REFERENCES users(id) ON DELETE CASCADE
- CHECK (target_amount > 0)
- CHECK (current_amount >= 0)
- CHECK (target_date > CURRENT_DATE)

### contributions

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| goal_id | uuid | No | - | FK to savings_goals |
| amount | DECIMAL(12,2) | No | - | Contribution amount |
| date | DATE | No | CURRENT_DATE | Contribution date |
| note | TEXT | Yes | NULL | Optional note |
| created_at | TIMESTAMP | No | now() | Creation time |

**Constraints:**
- FOREIGN KEY goal_id REFERENCES savings_goals(id) ON DELETE CASCADE
- CHECK (amount > 0)

### support_tickets

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | No | gen_random_uuid() | Primary key |
| user_id | uuid | No | - | FK to users |
| subject | VARCHAR(255) | No | - | Ticket subject |
| status | ENUM('OPEN', 'IN_PROGRESS', 'RESOLVED', 'CLOSED') | No | 'OPEN' | Ticket status |
| priority | ENUM('LOW', 'MEDIUM', 'HIGH') | No | 'MEDIUM' | Priority level |
| messages | JSONB | No | '[]' | Message thread (JSON array) |
| created_at | TIMESTAMP | No | now() | Creation time |
| updated_at | TIMESTAMP | No | now() | Last update |

**Constraints:**
- FOREIGN KEY user_id REFERENCES users(id) ON DELETE CASCADE

## Indexes

| Table | Index | Columns | Type | Purpose |
|-------|-------|---------|------|---------|
| users | users_email_idx | email | UNIQUE | Fast email lookup for login |
| users | users_status_idx | status | BTREE | Filter active users |
| categories | categories_user_idx | user_id, type | BTREE | Get user categories |
| transactions | transactions_user_date_idx | user_id, date | BTREE | Get user transactions by date |
| transactions | transactions_category_idx | category_id | BTREE | Get transactions by category |
| budgets | budgets_user_period_idx | user_id, period, start_date | BTREE | Get active budgets |
| savings_goals | goals_user_idx | user_id | BTREE | Get user goals |
| support_tickets | tickets_user_status_idx | user_id, status | BTREE | Get user tickets by status |

## Migrations

```bash
# Generate migration
npm run migration:generate -- -n MigrationName

# Run migrations
npm run migration:run

# Revert last migration
npm run migration:revert
```
