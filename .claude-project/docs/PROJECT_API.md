# API Reference: pennywise

## Base URL

- **Development**: `http://localhost:3000/api`
- **Production**: `https://api.pennywise.com`

## Authentication

This API uses **httpOnly cookie-based authentication** for secure session management.

### 🍪 For Browser/Web Clients (PRIMARY METHOD)

Authentication is handled automatically via httpOnly cookies:

**Flow:**
1. Client calls `POST /auth/login` with credentials
2. Backend validates and sets `accessToken` and `refreshToken` as httpOnly cookies via `Set-Cookie` header
3. All subsequent requests automatically include cookies (browser handles this)
4. Frontend uses `withCredentials: true` in axios/fetch configuration
5. **NO tokens stored in localStorage or sessionStorage**

**Cookie Configuration:**
```
HttpOnly: true       # JavaScript cannot access (XSS protection)
Secure: true         # HTTPS only (production)
SameSite: Strict     # CSRF protection (production)
Path: /
Max-Age: 86400       # 24 hours (access token)
```

**Frontend Setup:**
```javascript
// axios configuration
axios.create({
  baseURL: 'http://localhost:3000/api',
  withCredentials: true,  // REQUIRED to include cookies
})
```

**Backend CORS:**
```javascript
// NestJS main.ts
app.enableCors({
  origin: 'http://localhost:5173',
  credentials: true  // REQUIRED to allow cookies
})
```

### 🔑 For API Clients/External Services (FALLBACK)

Bearer token authentication is supported for non-browser clients:

```
Authorization: Bearer <token>
```

Obtain token via login endpoint, then pass in Authorization header for subsequent requests.

**When to use:**
- Mobile apps (non-web views)
- External API clients (Postman, curl, third-party)
- Server-to-server communication
- Command-line tools

**Note**: Web browsers should ALWAYS use cookies for security.

## Endpoints

### Auth

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/register` | Register new user | No |
| POST | `/auth/login` | Login user | No |
| POST | `/auth/refresh` | Refresh token | Yes |
| POST | `/auth/logout` | Logout user | Yes |

### Users

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/users` | List all users | Yes |
| GET | `/users/:id` | Get user by ID | Yes |
| PATCH | `/users/:id` | Update user | Yes |
| DELETE | `/users/:id` | Delete user | Yes |

### Budgets

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/budgets` | Get all user budgets | Yes |
| POST | `/budgets` | Create new budget | Yes |
| GET | `/budgets/:id` | Get budget details | Yes |
| PATCH | `/budgets/:id` | Update budget | Yes |
| DELETE | `/budgets/:id` | Delete budget | Yes |

### Transactions

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/transactions` | Get all transactions (with filters) | Yes |
| POST | `/transactions` | Create new transaction | Yes |
| GET | `/transactions/:id` | Get transaction details | Yes |
| PATCH | `/transactions/:id` | Update transaction | Yes |
| DELETE | `/transactions/:id` | Delete transaction | Yes |

### Goals

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/goals` | Get all savings goals | Yes |
| POST | `/goals` | Create new goal | Yes |
| GET | `/goals/:id` | Get goal details | Yes |
| PATCH | `/goals/:id` | Update goal | Yes |
| DELETE | `/goals/:id` | Delete goal | Yes |
| POST | `/goals/:id/contributions` | Add contribution to goal | Yes |

### Categories

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/categories` | Get all categories | Yes |
| POST | `/categories` | Create custom category | Yes |
| PATCH | `/categories/:id` | Update category | Yes |
| DELETE | `/categories/:id` | Delete custom category | Yes |

### Analytics

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/analytics/spending` | Get spending breakdown | Yes |
| GET | `/analytics/trends` | Get spending trends | Yes |
| GET | `/analytics/summary` | Get summary statistics | Yes |

### Admin

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/admin/users` | Get all users (admin) | Admin |
| PATCH | `/admin/users/:id/status` | Update user status | Admin |
| GET | `/admin/analytics` | Get platform analytics | Admin |
| GET | `/admin/support-tickets` | Get support tickets | Admin |
| PATCH | `/admin/support-tickets/:id` | Update ticket status | Admin |

## Request/Response Examples

### Login

**Request:**
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123",
  "rememberMe": false  // Optional: extends cookie expiration to 30 days
}
```

**Response Body:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "email": "user@example.com",
      "name": "John Doe",
      "role": "USER"
    }
  }
}
```

**Response Headers (Set-Cookie):**
```http
Set-Cookie: accessToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=86400
Set-Cookie: refreshToken=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=604800
```

**CRITICAL NOTES:**

1. **Tokens are NOT in response body** - only user data is returned
2. **Tokens are set via Set-Cookie headers** - browser stores automatically
3. **Frontend does NOT manually store tokens** - security vulnerability
4. **Cookies automatically included in subsequent requests**
5. **httpOnly flag prevents JavaScript access** - XSS protection

**Frontend Implementation:**
```typescript
// Login request
const response = await authService.login({
  email: 'user@example.com',
  password: 'password123'
});

// Backend sets cookies automatically via Set-Cookie header
// No localStorage.setItem() needed - in fact, DON'T do this!

// Dispatch user to Redux/state (NO tokens)
dispatch(loginSuccess(response.data.user));

// Navigate to dashboard
navigate('/dashboard');

// All subsequent API calls automatically include cookies
const data = await api.get('/protected-endpoint');
// No need to add Authorization header - cookies sent automatically
```

**Error Response:**
```json
{
  "success": false,
  "message": "Invalid credentials",
  "statusCode": 401
}
```

## Error Responses

| Status | Description |
|--------|-------------|
| 400 | Bad Request - Invalid input |
| 401 | Unauthorized - Invalid/missing token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 500 | Internal Server Error |
