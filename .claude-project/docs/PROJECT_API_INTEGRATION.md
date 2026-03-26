# API Integration Status - PennyWise

**Last Updated:** 2026-03-25
**Source:** PRD + HTML Prototypes + API Documentation

---

## Overview

This document tracks which frontend pages use which API endpoints, organized by user role and implementation status.

**Frontend Projects:**
- `frontend` - Main web application (User Individual role)
- `frontend` - Admin dashboard (Admin role)

**Status Values:**
- `NOT_STARTED` - Page/API not implemented
- `IN_PROGRESS` - Implementation underway
- `COMPLETED` - Fully implemented and tested
- `BLOCKED` - Waiting on dependency

---

## 1. Authentication Pages (Public)

| Route | HTML File | APIs Used | Params | Status |
|-------|-----------|-----------|--------|--------|
| `/` | `01-splash.html` | None (redirects to `/login`) | - | NOT_STARTED |
| `/login` | `02-login.html` | `POST /auth/login` | email, password, rememberMe | NOT_STARTED |
| `/signup` | `03-signup.html` | `POST /auth/register` | email, password, displayName, currency?, monthlyIncome? | NOT_STARTED |
| `/forgot-password` | `04-forgot-password.html` | `POST /auth/forgot-password` | email | NOT_STARTED |
| `/reset-password` | `05-reset-password.html` | `POST /auth/reset-password` | token, newPassword | NOT_STARTED |
| `/auth/google` | - | `GET /auth/google` | - | NOT_STARTED |
| `/auth/google/callback` | - | `GET /auth/google/callback` | code | NOT_STARTED |

### Additional Auth APIs
| API Endpoint | Method | Purpose | Used By |
|--------------|--------|---------|---------|
| `POST /auth/refresh` | POST | Refresh access token | All authenticated pages |
| `POST /auth/logout` | POST | Logout and clear cookies | All authenticated pages |

---

## 2. User Dashboard Pages

### 2.1 Dashboard (Home)

| Route | HTML File | APIs Used | Query Params | Status |
|-------|-----------|-----------|--------------|--------|
| `/dashboard` | `06-dashboard.html` | `GET /analytics/summary` | period (this_month, last_3_months) | NOT_STARTED |
|  |  | `GET /budgets?limit=5` | limit=5 (active budgets) | NOT_STARTED |
|  |  | `GET /transactions?limit=5` | limit=5 (recent transactions) | NOT_STARTED |
|  |  | `GET /goals` | - (all active goals) | NOT_STARTED |

#### Dashboard Components & Data Requirements
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Monthly Overview Card | `/analytics/summary` | totalIncome, totalExpenses, remainingBudget, spentPercentage |
| Budget Status Section | `/budgets?limit=5` | category.name, amount, spentAmount, percentage, status (green/yellow/red) |
| Recent Transactions | `/transactions?limit=5` | date, category.icon, description, amount, type |
| Savings Goals Summary | `/goals` | name, targetAmount, currentAmount, progressPercentage, targetDate |

### 2.2 Budgets

| Route | HTML File | APIs Used | Query Params | Status |
|-------|-----------|-----------|--------------|--------|
| `/budgets` | `07-budgets-list.html` | `GET /budgets` | - | NOT_STARTED |
|  |  | `POST /budgets` | categoryId, amount, period, startDate, endDate | NOT_STARTED |
| `/budgets/:id` | `08-budget-detail.html` | `GET /budgets/:id` | - | NOT_STARTED |
|  |  | `PATCH /budgets/:id` | amount?, period?, startDate?, endDate? | NOT_STARTED |
|  |  | `DELETE /budgets/:id` | - | NOT_STARTED |
|  |  | `GET /transactions?categoryId=X` | categoryId, startDate, endDate | NOT_STARTED |

#### Budget List Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Budget Cards | `/budgets` | category.name, amount, spentAmount, percentage, period, startDate, endDate |
| Create Budget Modal | `/categories` | id, name, icon (dropdown options) |
| Progress Bars | Calculated | percentage, color (green <80%, yellow 80-100%, red >100%) |

#### Budget Detail Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Budget Header | `/budgets/:id` | category.name, amount, period |
| Progress Visualization | `/budgets/:id` | spentAmount, amount, percentage |
| Transaction List | `/transactions?categoryId=X` | date, description, amount, note |
| Edit/Delete Actions | `/budgets/:id` | - |

### 2.3 Transactions

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/transactions` | `09-transactions-list.html` | `GET /transactions` | dateRange?, categoryId?, type? (INCOME/EXPENSE/ALL) | NOT_STARTED |
|  |  | `POST /transactions` | type, amount, categoryId, date, note? | NOT_STARTED |
| `/transactions/:id` | `10-transaction-detail.html` | `GET /transactions/:id` | - | NOT_STARTED |
|  |  | `PATCH /transactions/:id` | amount?, categoryId?, date?, note? | NOT_STARTED |
|  |  | `DELETE /transactions/:id` | - | NOT_STARTED |

#### Transaction List Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Transaction List | `/transactions` | date, category.icon, category.name, description, amount, type |
| Filter Options | - | Date range picker, category dropdown, type toggle |
| Add Transaction Modal | `/categories` | Categories for dropdown |
| Grouped by Date | Calculated | Transactions grouped by date |

#### Transaction Detail/Edit Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Transaction Details | `/transactions/:id` | type, amount, category, date, description, note |
| Edit Mode | `/transactions/:id` | Editable fields |
| Delete Action | `/transactions/:id` | Confirmation modal |

### 2.4 Savings Goals

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/goals` | `11-goals-list.html` | `GET /goals` | - | NOT_STARTED |
|  |  | `POST /goals` | name, targetAmount, targetDate | NOT_STARTED |
| `/goals/:id` | `12-goal-detail.html` | `GET /goals/:id` | - | NOT_STARTED |
|  |  | `PATCH /goals/:id` | name?, targetAmount?, targetDate? | NOT_STARTED |
|  |  | `DELETE /goals/:id` | - | NOT_STARTED |
|  |  | `POST /goals/:id/contributions` | amount, date, note? | NOT_STARTED |
|  |  | `GET /goals/:id/contributions` | - | NOT_STARTED |

#### Goals List Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Goal Cards | `/goals` | name, targetAmount, currentAmount, targetDate, progressPercentage |
| Create Goal Modal | - | Form inputs (name, targetAmount, targetDate) |
| Monthly Contribution | Calculated | Required monthly savings to reach goal |

#### Goal Detail Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Goal Header | `/goals/:id` | name, targetAmount, targetDate |
| Progress Visualization | `/goals/:id` | currentAmount, targetAmount, percentage |
| Current Amount Saved | `/goals/:id` | currentAmount |
| Days Remaining | Calculated | targetDate - today |
| Required Monthly Contribution | Calculated | (targetAmount - currentAmount) / months remaining |
| Contribution History | `/goals/:id/contributions` | date, amount, note |
| Add Contribution Button | - | Opens modal |

### 2.5 Analytics

| Route | HTML File | APIs Used | Query Params | Status |
|-------|-----------|-----------|--------------|--------|
| `/analytics` | `13-analytics.html` | `GET /analytics/spending` | period (this_month, last_3_months, last_6_months, custom) | NOT_STARTED |
|  |  | `GET /analytics/trends` | period | NOT_STARTED |
|  |  | `GET /analytics/summary` | period | NOT_STARTED |

#### Analytics Page Components
| Component | Data Source | Fields Needed | Chart Type |
|-----------|-------------|---------------|------------|
| Period Selector | - | Dropdown (This Month, Last 3 Months, Last 6 Months, Custom) | - |
| Spending Breakdown | `/analytics/spending` | category, amount, percentage | Pie Chart |
| Spending Trend | `/analytics/trends` | date, income, expense | Line Chart |
| Category Comparison | `/analytics/summary` | category, budgetAmount, actualAmount | Bar Chart |
| Summary Statistics | `/analytics/summary` | avgMonthlySpending, highestSpendingCategory, monthOverMonthChange | Stats Cards |

### 2.6 Settings

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/settings` | `14-settings.html` | `GET /users/me` | - | NOT_STARTED |
|  |  | `PATCH /users/me` | displayName?, currency?, monthlyIncome? | NOT_STARTED |
|  |  | `GET /categories` | type=CUSTOM (user's custom categories) | NOT_STARTED |
|  |  | `POST /categories` | name, icon | NOT_STARTED |
|  |  | `PATCH /categories/:id` | name?, icon?, status? | NOT_STARTED |
|  |  | `DELETE /categories/:id` | - | NOT_STARTED |
|  |  | `POST /auth/change-password` | currentPassword, newPassword | NOT_STARTED |
|  |  | `POST /users/export` | - (returns CSV) | NOT_STARTED |
|  |  | `DELETE /users/me` | - (with confirmation) | NOT_STARTED |

#### Settings Page Sections
| Section | Data Source | Fields Needed |
|---------|-------------|---------------|
| Profile Section | `/users/me` | displayName (editable), email (read-only) |
| Preferences Section | `/users/me` | currency (dropdown), monthlyIncome (editable) |
| Categories Section | `/categories?type=CUSTOM` | name, icon, status (View/Edit/Delete custom categories, Add new) |
| Notifications Section | Frontend State | Budget alert toggle, Weekly summary toggle |
| Account Section | - | Change password, Export data (CSV), Delete account |
| About Section | - | App version, Terms of Service, Privacy Policy |

---

## 3. Admin Dashboard Pages

### 3.1 Admin Dashboard (Home)

| Route | HTML File | APIs Used | Query Params | Status |
|-------|-----------|-----------|--------------|--------|
| `/admin/dashboard` | `15-admin-dashboard.html` | `GET /admin/analytics` | period (today, last_7_days, last_30_days, custom) | NOT_STARTED |

#### Admin Dashboard Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Statistics Cards | `/admin/analytics` | totalUsers, activeUsers (last 30 days), totalTransactions (this month), totalBudgetAmount |
| Period Filter | - | Dropdown (Today, Last 7 days, Last 30 days, Custom) |
| User Signups Trend | `/admin/analytics` | date, newUsersCount | Line Chart |
| Transactions by Category | `/admin/analytics` | category, transactionCount | Bar Chart |
| Recent Activity | `/admin/analytics` | recentUsers (last 10), recentSupportTickets | Lists |

### 3.2 User Management

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/admin/users` | `16-user-management.html` | `GET /admin/users` | search?, status? (ACTIVE/SUSPENDED), dateRange? | NOT_STARTED |
|  |  | `PATCH /admin/users/:id/status` | status (ACTIVE/SUSPENDED) | NOT_STARTED |
|  |  | `DELETE /admin/users/:id` | - | NOT_STARTED |
|  |  | `POST /admin/users/:id/reset-password` | - (sends reset email) | NOT_STARTED |

#### User Management Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Search Bar | - | Keyword search (name, email, user ID) |
| Filters | - | Status dropdown, Date range picker (signup date) |
| Export Button | - | Download user list CSV |
| Bulk Actions | - | Suspend / Activate / Delete (selected users) |
| User Table | `/admin/users` | userId, displayName, email, status, signupDate, lastLogin |
| Action Column | - | View / Suspend / Delete buttons per row |
| Pagination | - | Items per page selector (10/25/50/100) |
| Detail Drawer | `/admin/users/:id` | All user info, total transactions, active budgets, active goals |

#### Detail Drawer (Per User)
| Section | Data | Actions |
|---------|------|---------|
| Account Info | displayName, email, userId, status | - |
| Account Statistics | totalTransactions, activeBudgetsCount, activeGoalsCount | - |
| Account Actions | - | Activate/Suspend, Reset Password (email), Delete |
| Timestamps | createdAt, lastLogin, lastModified | - |

### 3.3 Category Management

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/admin/categories` | `17-category-management.html` | `GET /categories` | type? (SYSTEM/CUSTOM), status? (ACTIVE/INACTIVE) | NOT_STARTED |
|  |  | `POST /categories` | name, icon, type=SYSTEM | NOT_STARTED |
|  |  | `PATCH /categories/:id` | name?, icon?, status? | NOT_STARTED |
|  |  | `DELETE /categories/:id` | - (only CUSTOM categories) | NOT_STARTED |

#### Category Management Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Search Bar | - | Category name search |
| Filters | - | Type (System/Custom), Status (Active/Inactive) |
| Create Category Button | - | Opens creation modal |
| Category Table | `/categories` | categoryId, name, icon, type (System/Custom), status, usageCount (# transactions) |
| Action Column | - | Edit / Deactivate / Delete (custom only) |
| Pagination | - | Items per page selector |

**Note:** System categories cannot be deleted, only deactivated. Custom categories can be deleted only if `usageCount = 0`.

### 3.4 Platform Analytics

| Route | HTML File | APIs Used | Query Params | Status |
|-------|-----------|-----------|--------------|--------|
| `/admin/analytics` | `18-platform-analytics.html` | `GET /admin/analytics` | period (today, last_7_days, last_30_days, last_90_days, custom) | NOT_STARTED |

#### Platform Analytics Page Components
| Component | Data Source | Fields Needed | Chart Type |
|-----------|-------------|---------------|------------|
| Period Selector | - | Dropdown + Custom date range | - |
| User Analytics | `/admin/analytics` | totalRegistered, DAU, MAU, retentionRate, newVsReturning | Stats + Chart |
| Transaction Analytics | `/admin/analytics` | totalTransactions, avgTransactionsPerUser, transactionsByCategoryBreakdown, volumeTrend | Stats + Pie + Line |
| Budget Analytics | `/admin/analytics` | totalBudgetAmountTracked, avgBudgetPerUser, mostCommonBudgetCategories, complianceRate | Stats + Bar |
| Export Options | - | Download analytics report (CSV/Excel), Select metrics, Date range | - |

### 3.5 Support Tickets

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/admin/support-tickets` | `19-support-tickets.html` | `GET /admin/support-tickets` | search?, status? (OPEN/IN_PROGRESS/RESOLVED/CLOSED), priority? (LOW/MEDIUM/HIGH) | NOT_STARTED |
|  |  | `PATCH /admin/support-tickets/:id` | status?, priority? | NOT_STARTED |
|  |  | `POST /admin/support-tickets/:id/messages` | message | NOT_STARTED |
|  |  | `DELETE /admin/support-tickets/:id` | - | NOT_STARTED |

#### Support Tickets Page Components
| Component | Data Source | Fields Needed |
|-----------|-------------|---------------|
| Search Bar | - | Keyword search (ticket ID, user email, subject) |
| Filters | - | Status dropdown, Priority dropdown, Date range |
| Bulk Actions | - | Mark as Resolved / Close / Delete (selected tickets) |
| Ticket Table | `/admin/support-tickets` | ticketId, user (name, email), subject, status, priority, createdDate, lastUpdated |
| Action Column | - | View / Update Status buttons |
| Detail Drawer | `/admin/support-tickets/:id` | All ticket info + message thread |

#### Ticket Detail Drawer
| Section | Data | Actions |
|---------|------|---------|
| Ticket Info | ID, subject, status, priority, createdDate | - |
| User Info | Name, email, userId | - |
| Message Thread | Original message, Admin responses, Reply input field | - |
| Actions | - | Update status dropdown, Update priority dropdown, Close ticket |

### 3.6 Admin Settings

| Route | HTML File | APIs Used | Query Params/Body | Status |
|-------|-----------|-----------|-------------------|--------|
| `/admin/settings` | `20-admin-settings.html` | `GET /admin/settings` | - | NOT_STARTED |
|  |  | `PATCH /admin/settings` | defaultCurrency?, sessionTimeout? | NOT_STARTED |

#### Admin Settings Page Sections
| Section | Data Source | Fields Needed |
|---------|-------------|---------------|
| System Settings | `/admin/settings` | defaultCurrency (for new users), sessionTimeout |
| Notification Settings | `/admin/settings` | Enable/disable system-wide push notifications, Configure notification templates |
| Security Settings | `/admin/settings` | Password policy configuration, Session management |

---

## 4. API Service Files (Frontend Implementation)

| Service | Location | Endpoints Covered |
|---------|----------|-------------------|
| **authService.ts** | `frontend/src/services/` | login, register, logout, refresh, forgotPassword, resetPassword, googleAuth |
| **userService.ts** | `frontend/src/services/` | getProfile, updateProfile, changePassword, exportData, deleteAccount |
| **budgetService.ts** | `frontend/src/services/` | getBudgets, createBudget, getBudget, updateBudget, deleteBudget |
| **transactionService.ts** | `frontend/src/services/` | getTransactions, createTransaction, getTransaction, updateTransaction, deleteTransaction |
| **goalService.ts** | `frontend/src/services/` | getGoals, createGoal, getGoal, updateGoal, deleteGoal, addContribution, getContributions |
| **categoryService.ts** | `frontend/src/services/` | getCategories, createCategory, updateCategory, deleteCategory |
| **analyticsService.ts** | `frontend/src/services/` | getSpending, getTrends, getSummary |
| **adminService.ts** | `frontend/src/services/` | getUsers, updateUserStatus, deleteUser, getAdminAnalytics, getCategories, updateCategory, getSupportTickets, updateTicket |

---

## 5. Common Integration Patterns

### 5.1 Axios Instance Configuration

```typescript
// src/config/axios.ts
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api',
  withCredentials: true,  // CRITICAL: Required for cookie-based auth
  headers: {
    'Content-Type': 'application/json',
  },
});

// Response interceptor for token refresh
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        await api.post('/auth/refresh');
        return api(originalRequest);
      } catch (refreshError) {
        // Redirect to login
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);

export default api;
```

### 5.2 Standard API Response Format

```typescript
// Success response
{
  "success": true,
  "data": { ... },
  "message": "Optional success message"
}

// Error response
{
  "success": false,
  "message": "Error message",
  "statusCode": 400,
  "errors": [...]  // Optional validation errors
}
```

### 5.3 Loading & Error States

All pages must implement:
- **Loading State**: Show skeleton/spinner while fetching
- **Error State**: Show error message with retry button
- **Empty State**: Show helpful message when no data exists

---

## 6. Implementation Checklist

### Global Setup
- [ ] Set up Axios instance with `withCredentials: true`
- [ ] Configure auth interceptors for token refresh
- [ ] Implement global error handling
- [ ] Implement global loading states
- [ ] Add TypeScript types for all API responses

### Per Page
- [ ] Create API service methods
- [ ] Implement loading states
- [ ] Implement error states
- [ ] Implement empty states
- [ ] Add form validation
- [ ] Test all CRUD operations
- [ ] Test filters and search
- [ ] Test pagination (where applicable)

### Testing
- [ ] Test authentication flow (login, logout, refresh)
- [ ] Test role-based access (USER vs ADMIN)
- [ ] Test API error scenarios (401, 403, 404, 500)
- [ ] Test network failures
- [ ] Test concurrent requests
- [ ] Test real-time data updates

---

## 7. Missing/Unclear Requirements

The following features mentioned in PRD need clarification:

| Feature | PRD Reference | Question | Status |
|---------|---------------|----------|--------|
| Transaction Import (CSV) | Additional Questions #3 | Is CSV import needed? | Pending Clarification |
| Default Expense Categories | Additional Questions #4 | Need specific list of system categories | Pending Clarification |
| Weekly Summary Email | Recommended Questions #1 | Email content format? | Pending Clarification |
| Multi-device Sync | Recommended Questions #4 | Real-time notifications needed? | Pending Clarification |
| Recurring Transactions | Recommended Questions #3 | Support for recurring bills? | Pending Clarification |

---

**Last Review:** 2026-03-25
**Next Review:** After initial frontend implementation
