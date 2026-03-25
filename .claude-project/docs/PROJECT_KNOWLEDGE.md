# Project Knowledge: pennywise

## Overview

PennyWise is a personal budgeting web application designed for users who want to track their income, expenses, and savings goals. The app provides a simple, intuitive interface with visual analytics to help users manage their finances effectively without complex financial jargon.

### Goals

1. Help users create and maintain monthly budgets with ease
2. Provide clear visual insights into spending patterns and trends
3. Enable users to set and track progress toward savings goals
4. Deliver timely alerts to prevent overspending

### User Types

| Role | Permissions |
|------|-------------|
| **User (Individual)** | Can create personal account, set monthly budgets, track income and expenses, categorize transactions, view spending analytics, set savings goals, receive budget alerts |
| **Admin** | Full system management, user management, view platform analytics, manage expense categories, handle support tickets |

### Terminology

| Term | Definition |
|------|------------|
| Budget | A spending limit set for a specific category over a time period |
| Transaction | A single income or expense entry |
| Category | A classification for transactions (e.g., Food, Transport) |
| Savings Goal | A target amount to save by a specific date |
| Remaining Budget | The amount left to spend in a budget category |
| Overspending | When expenses exceed the set budget for a category |
| Income | Money received (salary, gifts, refunds, etc.) |

### External Services

| Service | Purpose | Documentation |
|---------|---------|---------------|
| Google OAuth | Social login for easy account creation | [Google Identity](https://developers.google.com/identity) |
| Firebase Cloud Messaging | Push notifications for budget alerts | [FCM Docs](https://firebase.google.com/docs/cloud-messaging) |

## Tech Stack

- **Backend**: nestjs
- **Frontend**: react
- **Database**: PostgreSQL 16
- **Cache**: Redis 7
- **Deployment**: Docker Compose

## Architecture

```
pennywise/
├── backend/           # API server
├── frontend/          # Web application
├── frontend-dashboard/ # Admin dashboard (if applicable)
├── mobile/            # Mobile app (if applicable)
└── docker-compose.yml
```

## Key Decisions

| Decision | Rationale | Date |
|----------|-----------|------|
| [Decision 1] | [Why this choice was made] | YYYY-MM-DD |

## Development Setup

```bash
# Clone with submodules
git clone --recurse-submodules <repo-url>

# Start services
docker-compose up -d
```

## Environment Variables

### Backend (.env)

| Variable | Description | Required | Default | Example |
|----------|-------------|----------|---------|---------|
| `DATABASE_URL` | Database connection string | Yes | - | `postgresql://user:pass@localhost:5432/db` |
| `AUTH_JWT_SECRET` | JWT signing secret (use strong random string) | Yes | - | `your-secure-secret-key-min-32-chars` |
| `AUTH_TOKEN_COOKIE_NAME` | Access token cookie name | No | `accessToken` | `accessToken` |
| `AUTH_TOKEN_EXPIRE_TIME` | Access token expiration | No | `24h` | `24h`, `1d`, `3600s` |
| `AUTH_TOKEN_EXPIRED_TIME_REMEMBER_ME` | Extended expiration for "remember me" | No | `30d` | `30d`, `720h` |
| `AUTH_REFRESH_TOKEN_COOKIE_NAME` | Refresh token cookie name | No | `refreshToken` | `refreshToken` |
| `AUTH_REFRESH_TOKEN_EXPIRE_TIME` | Refresh token expiration | No | `7d` | `7d`, `168h` |
| `FRONTEND_URL` | Frontend URL for CORS allowlist | Yes | `http://localhost:5173` | `https://app.example.com` |
| `MODE` | Environment mode (affects cookie security) | Yes | `DEV` | `DEV`, `PROD` |

### Frontend (.env)

| Variable | Description | Required | Default | Example |
|----------|-------------|----------|---------|---------|
| `VITE_API_URL` | Backend API base URL | Yes | `http://localhost:3000/api` | `https://api.example.com` |

### Cookie Security Configuration

**Automatically configured based on `MODE` environment variable:**

| Setting | Development (`MODE=DEV`) | Production (`MODE=PROD`) |
|---------|--------------------------|--------------------------|
| `httpOnly` | `true` | `true` |
| `secure` | `false` | `true` (HTTPS only) |
| `sameSite` | `'lax'` | `'strict'` |
| `path` | `'/'` | `'/'` |

**Cookie Expiration:**
- **Access Token**: 24 hours (or extended to 30 days with "Remember Me")
- **Refresh Token**: 7 days

**Security Notes:**

1. **httpOnly Flag**: Prevents JavaScript access to cookies (XSS protection)
2. **Secure Flag**: Ensures cookies only sent over HTTPS in production
3. **SameSite Policy**: Prevents CSRF attacks (`strict` in production, `lax` in dev for easier testing)
4. **Short-lived Access Tokens**: Reduces exposure window if token compromised
5. **Long-lived Refresh Tokens**: Enables automatic token refresh without re-login

### Authentication Environment Variables Explained

**`AUTH_JWT_SECRET`:**
- Used to sign JWT tokens
- Must be a strong, random string (minimum 32 characters recommended)
- NEVER commit this to version control
- Use different secrets for dev/staging/production

**`AUTH_TOKEN_EXPIRE_TIME`:**
- How long access token remains valid
- Shorter = more secure but more frequent refreshes
- Recommended: 15min-24h range
- Format: `1h`, `24h`, `1d`, `86400s`

**`AUTH_REFRESH_TOKEN_EXPIRE_TIME`:**
- How long refresh token remains valid
- Longer = less frequent re-logins needed
- Recommended: 7d-30d range
- User must re-login after this expires

**`MODE`:**
- Controls cookie security flags
- `DEV`: Allows http, relaxed sameSite for local development
- `PROD`: Enforces https, strict sameSite for production security

**`FRONTEND_URL`:**
- CORS allowlist for cookie-based auth
- Must match exact origin (protocol + domain + port)
- Multiple origins: Use comma-separated list or array

## Security Architecture

### Authentication Security Model

This project uses **httpOnly cookie-based authentication** to prevent XSS token theft.

#### Why httpOnly Cookies Over localStorage?

| Attack Vector | localStorage | httpOnly Cookie | Winner |
|---------------|--------------|-----------------|--------|
| **XSS (Cross-Site Scripting)** | ❌ VULNERABLE - JS can access tokens | ✅ PROTECTED - JS cannot access | Cookie |
| **CSRF (Cross-Site Request Forgery)** | ✅ Not applicable | ⚠️ Possible (mitigated with SameSite) | Tie with mitigation |
| **Man-in-the-Middle** | ❌ Vulnerable without HTTPS | ✅ Protected with Secure flag | Cookie |
| **Token Theft via DevTools** | ❌ Visible in Application tab | ✅ Hidden from JavaScript | Cookie |

**Verdict**: httpOnly cookies are significantly more secure for web applications.

#### Security Features Implemented

1. **httpOnly Cookies**
   - Tokens inaccessible to JavaScript
   - Prevents XSS token theft
   - Automatic browser management

2. **Secure Flag (Production)**
   - Cookies only sent over HTTPS
   - Prevents man-in-the-middle token interception
   - Automatically enabled when `MODE=PROD`

3. **SameSite Policy**
   - `strict` in production: Blocks all cross-site requests
   - `lax` in development: Allows top-level navigation
   - Prevents CSRF attacks

4. **Short-lived Access Tokens**
   - 24-hour expiration (default)
   - Reduces exposure window if compromised
   - Automatic refresh via refresh token

5. **Long-lived Refresh Tokens**
   - 7-day expiration (default)
   - Enables seamless token refresh
   - Stored as httpOnly cookie

6. **CORS with Credentials**
   - Explicit origin allowlist
   - Credentials required for cookie transmission
   - Prevents unauthorized cross-origin requests

#### Threat Model & Mitigations

| Threat | Mitigation |
|--------|------------|
| XSS injects malicious script | httpOnly cookies prevent token access |
| CSRF forces unwanted actions | SameSite policy blocks cross-site requests |
| MITM intercepts tokens | Secure flag + HTTPS enforcement |
| Token stolen from localStorage | Tokens never stored in localStorage |
| Replay attack with old token | Short-lived tokens with expiration |
| Session hijacking | Token refresh rotation + device tracking (optional) |

#### Security Best Practices

**✅ DO:**
- Use httpOnly cookies for all authentication tokens
- Enable Secure flag in production (HTTPS)
- Use SameSite=Strict in production
- Implement short-lived access tokens (15min-24h)
- Implement automatic token refresh
- Log and monitor authentication failures
- Use HTTPS in production
- Rotate JWT secret regularly

**❌ DON'T:**
- Store tokens in localStorage or sessionStorage
- Return tokens in response body (use Set-Cookie headers)
- Disable httpOnly flag
- Use long-lived access tokens (>24h)
- Ignore CORS configuration
- Use SameSite=None without good reason
- Allow credentials from all origins (*)

#### Compliance Considerations

- **GDPR**: Cookies require user consent in EU
- **OWASP Top 10**: Mitigates A02:2021 (Cryptographic Failures), A07:2021 (Identification and Authentication Failures)
- **PCI DSS**: Supports secure authentication requirements
- **SOC 2**: Demonstrates security controls for authentication

#### Further Reading

- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [MDN: Using HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [OWASP: Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)

---

## External Services

| Service | Purpose | Documentation |
|---------|---------|---------------|
| [Service 1] | [What it's used for] | [Link] |
