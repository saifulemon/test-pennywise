# Frontend Auth Flow - PennyWise

**Last Updated:** 2026-03-25
**Source:** PRD (prd.pdf) + HTML Prototypes (`.claude-project/resources/HTML/auth/`)

---

## Overview

PennyWise uses **httpOnly cookie-based authentication** for security. All auth flows involve:
- Backend setting cookies via `Set-Cookie` headers
- Frontend using `withCredentials: true` for axios
- No tokens stored in localStorage/sessionStorage

---

## 1. Splash Screen Flow

### Screen Details
- **File**: `01-splash.html`
- **Route**: `/` (or `/splash`)
- **Design**:
  - App logo centered with emerald green (#10B981) background
  - Auto-redirect to Login page after 2 seconds

### Behavior
1. User lands on splash page
2. App logo displays with brand colors
3. After 2 seconds → Redirect to Login (`/login`)

---

## 2. Login Flow

### Login Page
- **File**: `02-login.html`
- **Route**: `/login`

### Form Fields

| Field | Type | Placeholder | Validation | Required |
|-------|------|-------------|------------|----------|
| Email | `email` | "Enter your email" | Email format | Yes |
| Password | `password` | "Enter your password" | Min 8 characters | Yes |

### UI Components
- **Left Side (Desktop)**: Brand illustration with emerald gradient background, abstract chart visual
- **Right Side**: Login form card
- **Social Login**: Google OAuth button
- **Links**:
  - "Forgot Password?" → `/forgot-password`
  - "Don't have an account? Sign up" → `/signup`

### Validation Rules
- **Email**: Valid email format required
- **Password**: Minimum 8 characters
- **On Error**: Display inline error message below field

### Submit Login - API Call

```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123",
  "rememberMe": false  // Optional checkbox
}
```

### On Success
1. Backend sets `accessToken` and `refreshToken` as httpOnly cookies via `Set-Cookie` header
2. Response body contains user data (NO tokens):
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "displayName": "John Doe",
      "role": "USER" // or "ADMIN"
    }
  }
}
```
3. Frontend stores user data in state/Redux (NOT tokens)
4. Redirect based on role:
   - **USER** → `/dashboard` (user dashboard)
   - **ADMIN** → `/admin/dashboard` (admin dashboard)

### On Failure
1. Stay on Login Page
2. Display error message below form: "Invalid credentials" (inline error)

---

## 3. Sign Up Flow

### Sign Up Page
- **File**: `03-signup.html`
- **Route**: `/signup`

### Form Fields

| Field | Type | Placeholder | Validation | Required | Default |
|-------|------|-------------|------------|----------|---------|
| Email | `email` | "Enter your email" | Unique email, valid format | Yes | - |
| Password | `password` | "Create password" | Min 8 characters | Yes | - |
| Display Name | `text` | "Your name" | Min 2 characters | Yes | - |
| Currency | `select` | "Select currency" | ISO 4217 code | No | USD |
| Monthly Income | `number` | "Optional" | Positive number | No | NULL |

### UI Components
- **Left Side (Desktop)**: Similar brand illustration as login
- **Right Side**: Sign up form card
- **Social Signup**: Google OAuth button
- **Links**: "Already have an account? Login" → `/login`

### Validation Rules
- **Email**: Must be unique in system
- **Password**: Minimum 8 characters
- **Display Name**: Minimum 2 characters
- **Currency**: Optional, defaults to USD
- **Monthly Income**: Optional, for budget suggestions

### Submit Sign Up - API Call

```http
POST /api/auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123",
  "displayName": "John Doe",
  "currency": "USD",
  "monthlyIncome": 5000  // Optional
}
```

### On Success
1. Account created with `role: 'USER'`, `status: 'ACTIVE'`
2. Backend sets auth cookies (same as login)
3. Response returns user data
4. Redirect to `/dashboard`

### On Failure
1. Stay on Sign Up Page
2. Display error: "Email already exists" or validation error

---

## 4. Forgot Password Flow

### Forgot Password Page
- **File**: `04-forgot-password.html`
- **Route**: `/forgot-password`

### Form Fields

| Field | Type | Placeholder | Required |
|-------|------|-------------|----------|
| Email | `email` | "Enter your email" | Yes |

### Submit Flow
1. User enters email
2. System sends password reset link to email
3. Show success message: "Password reset link sent to your email"
4. Link to "Back to Login" → `/login`

### API Call

```http
POST /api/auth/forgot-password
Content-Type: application/json

{
  "email": "user@example.com"
}
```

---

## 5. Reset Password Flow

### Reset Password Page
- **File**: `05-reset-password.html`
- **Route**: `/reset-password?token=<reset_token>`

### Form Fields

| Field | Type | Placeholder | Validation | Required |
|-------|------|-------------|------------|----------|
| New Password | `password` | "Enter new password" | Min 8 characters | Yes |
| Confirm Password | `password` | "Confirm password" | Must match new password | Yes |

### Validation Rules
- **New Password**: Minimum 8 characters
- **Confirm Password**: Must match new password exactly

### Submit Flow
1. User enters new password + confirmation
2. Password validation rules apply
3. On success: Redirect to Login with success message

### API Call

```http
POST /api/auth/reset-password
Content-Type: application/json

{
  "token": "<reset_token_from_url>",
  "newPassword": "newpassword123"
}
```

### On Success
1. Redirect to `/login`
2. Show success message: "Password reset successful. Please login."

---

## 6. Google OAuth Flow

### OAuth Button
- Displayed on both Login and Sign Up pages
- Button text: "Continue with Google"
- Icon: Google logo

### Flow
1. User clicks "Continue with Google"
2. Redirect to Google OAuth consent screen
3. User authorizes app
4. Google redirects back to `/auth/google/callback?code=<auth_code>`
5. Backend exchanges code for user info
6. Backend creates account (if new user) or logs in (existing user)
7. Backend sets auth cookies
8. Redirect to `/dashboard`

### API Endpoints

```http
GET /api/auth/google
# Redirects to Google OAuth consent screen

GET /api/auth/google/callback?code=<code>
# Backend handles OAuth callback
# Sets cookies and returns user data
```

---

## 7. Session Management

### Cookie Configuration
- **Access Token Cookie**: `accessToken`
  - **Expiration**: 24 hours (or 30 days with "Remember Me")
  - **HttpOnly**: true (prevents JavaScript access)
  - **Secure**: true (production only, HTTPS required)
  - **SameSite**: strict (production) / lax (development)

- **Refresh Token Cookie**: `refreshToken`
  - **Expiration**: 7 days
  - **HttpOnly**: true
  - **Secure**: true (production)
  - **SameSite**: strict (production) / lax (development)

### Auto Refresh Flow
1. Access token expires after 24 hours
2. Frontend intercepts 401 response
3. Frontend calls `POST /api/auth/refresh` (refresh token sent automatically via cookie)
4. Backend validates refresh token
5. Backend issues new access token via `Set-Cookie`
6. Retry original request

### Axios Configuration

```typescript
// Frontend axios setup
const api = axios.create({
  baseURL: 'http://localhost:3000/api',
  withCredentials: true,  // REQUIRED for cookies
});

// Response interceptor for token refresh
api.interceptors.response.use(
  response => response,
  async error => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      await api.post('/auth/refresh');
      return api(error.config);
    }
    return Promise.reject(error);
  }
);
```

---

## 8. Logout Flow

### Logout Action
- **Trigger**: User clicks "Logout" in navigation menu or settings
- **Route**: Can be triggered from any authenticated page

### API Call

```http
POST /api/auth/logout
# Cookies automatically sent via withCredentials
```

### On Success
1. Backend clears `accessToken` and `refreshToken` cookies
2. Frontend clears user state
3. Redirect to `/login`

---

## 9. Route Protection

### Protected Routes (Require Authentication)

| Route | Required Role | Redirect If Unauthenticated |
|-------|---------------|----------------------------|
| `/dashboard` | USER | `/login` |
| `/budgets` | USER | `/login` |
| `/transactions` | USER | `/login` |
| `/goals` | USER | `/login` |
| `/analytics` | USER | `/login` |
| `/settings` | USER | `/login` |
| `/admin/*` | ADMIN | `/login` |

### Public Routes

| Route | Description |
|-------|-------------|
| `/` | Splash screen |
| `/login` | Login page |
| `/signup` | Sign up page |
| `/forgot-password` | Forgot password |
| `/reset-password` | Reset password |

### Role-Based Access
- **USER Role**: Access to user dashboard, budgets, transactions, goals, analytics, settings
- **ADMIN Role**: Access to admin dashboard, user management, category management, platform analytics, support tickets

### Implementation Pattern

```typescript
// React Router example
<Route
  path="/dashboard"
  element={
    <RequireAuth allowedRoles={['USER', 'ADMIN']}>
      <Dashboard />
    </RequireAuth>
  }
/>

<Route
  path="/admin/*"
  element={
    <RequireAuth allowedRoles={['ADMIN']}>
      <AdminRoutes />
    </RequireAuth>
  }
/>
```

---

## 10. User Navigation Menu

### User (Individual) Navigation
From PRD Section 2.1:
1. Dashboard
2. Budgets
3. Transactions
4. Goals
5. Analytics
6. Settings

### Admin Navigation
From PRD Part 3:
1. Dashboard (Admin Home)
2. User Management
3. Category Management
4. Platform Analytics
5. Support Tickets
6. Settings

---

## Key Files (Implementation)

| Purpose | Path |
|---------|------|
| Login Page | `frontend/src/pages/auth/Login.tsx` |
| Sign Up Page | `frontend/src/pages/auth/SignUp.tsx` |
| Forgot Password | `frontend/src/pages/auth/ForgotPassword.tsx` |
| Reset Password | `frontend/src/pages/auth/ResetPassword.tsx` |
| Auth Service | `frontend/src/services/authService.ts` |
| Auth Context/State | `frontend/src/context/AuthContext.tsx` |
| Auth Types | `frontend/src/types/auth.types.ts` |
| Route Guards | `frontend/src/components/auth/RequireAuth.tsx` |
| Routes Config | `frontend/src/router/index.tsx` |
| HTML Prototypes | `.claude-project/resources/HTML/auth/` |

---

## Security Notes

1. **Never store tokens in localStorage** - use httpOnly cookies only
2. **Always use `withCredentials: true`** - required for cookie transmission
3. **Backend CORS must allow credentials** - `credentials: true`
4. **Password hashing** - bcrypt with salt rounds 10+ on backend
5. **Input validation** - both frontend and backend validation required
