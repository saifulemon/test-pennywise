# Frontend Auth Flow: pennywise

> Fill each section by reading:
> - **PRD**: `.claude-project/prd/`
> - **HTML Prototypes**: `.claude-project/resources/HTML/`

---

## Login Flow

### Step 1: Login Page

[Extract from PRD and HTML login screen]

- **Route**: [from resources]
- **Form fields**: [list all fields from HTML prototype with type, placeholder, validation]
- **UI**: [describe layout from HTML prototype]
- **Links/actions**: [forgot password, sign up link, social login — if exists in resources]

### Step 2: Submit Login

[Extract success/failure behavior from PRD]

- **If Success**:
  1. [What happens on success — from PRD]
  2. [Where user is redirected — from PRD, per role if applicable]

- **If Not Success**:
  1. Stay on Login Page
  2. [How error is shown — from HTML prototype]

---

## Sign Up Flow

[Extract from PRD and HTML sign-up screen. If no sign-up exists in PRD, document the alternative (e.g., admin-provisioned, invitation-only)]

### Step 1: Sign Up Page

- **Route**: [from resources]
- **Form fields**: [list all fields from HTML prototype with type, placeholder, validation]
- **UI**: [describe layout from HTML prototype]
- **Links/actions**: [already have account, terms — if exists in resources]

### Step 2: Submit Sign Up

- **If Account Created Successfully**:
  1. [What happens — from PRD]
  2. [Where user is redirected — from PRD]

- **If Account Creation Failed**:
  1. Stay on Sign Up Page
  2. [How error is shown — from HTML prototype]

---

## Additional Auth Steps

[Extract from PRD — include only what exists. Delete sections that don't apply]

- **Forgot Password**: [steps if exists in PRD]
- **Email Verification**: [steps if exists in PRD]
- **OAuth / Social Login**: [steps if exists in PRD]
- **Password Change**: [steps if exists in PRD]

---

## Session & Route Protection

[Extract from PRD]

- **Session management**: [how sessions work — from PRD]
- **Protected routes**: [which routes require auth — from PRD]
- **Role-based access**: [which roles access which pages — from PRD]
- **Logout**: [logout behavior — from PRD]

---

## Key Files

[Fill after implementation]

| Purpose | Path |
|---------|------|
| Login Page | |
| Sign Up Page | |
| Auth Service | |
| Auth State | |
| Auth Types | |
| Validation | |
| Route Guards | |
| Routes Config | |
| HTML Prototype | `.claude-project/resources/HTML/auth/` |
