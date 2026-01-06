# OAuth 2.0 - Multiple Providers Integration

## Objective
Implement OAuth 2.0 authentication supporting multiple providers (Google, GitHub, Facebook) in a full-stack application with session management.

## Technology Stack
- Frontend: React 18+ with TypeScript
- Backend: Node.js/Express with TypeScript
- Passport.js for OAuth strategies
- express-session for session management
- Redis for session storage
- PostgreSQL/MongoDB for user data

## Requirements

### 1. Supported OAuth Providers
- Google OAuth 2.0
- GitHub OAuth 2.0
- Facebook OAuth 2.0
- Extensible architecture for adding more providers

### 2. Backend Authentication Endpoints
- **GET /api/auth/google** - Initiate Google OAuth flow
- **GET /api/auth/google/callback** - Google OAuth callback
- **GET /api/auth/github** - Initiate GitHub OAuth flow
- **GET /api/auth/github/callback** - GitHub OAuth callback
- **GET /api/auth/facebook** - Initiate Facebook OAuth flow
- **GET /api/auth/facebook/callback** - Facebook OAuth callback
- **GET /api/auth/logout** - Logout and destroy session
- **GET /api/auth/user** - Get current authenticated user
- **POST /api/auth/link/:provider** - Link additional OAuth provider to existing account
- **DELETE /api/auth/unlink/:provider** - Unlink OAuth provider

### 3. Database Schema
Users table/collection:
- id (UUID/ObjectId)
- email (unique)
- displayName
- avatar
- providers (array of linked providers)
- googleId (nullable)
- githubId (nullable)
- facebookId (nullable)
- createdAt
- updatedAt
- lastLogin

### 4. Session Management
- Redis-backed session store for scalability
- Secure, httpOnly cookies
- Session expiration (7 days)
- Session refresh on activity
- CSRF protection

### 5. Frontend Components
Create the following:
- **OAuth Login Page** - Buttons for each provider
- **Account Linking UI** - Manage connected providers
- **User Profile** - Display user info from OAuth providers
- **Protected Route Wrapper** - Check authentication status

### 6. Features
- Account merging when same email is used across providers
- Link multiple OAuth providers to one account
- Unlink providers (with validation that at least one remains)
- Fetch and store user profile data from each provider
- Handle OAuth errors gracefully
- Automatic redirect after authentication
- Remember return URL before OAuth flow
- Profile picture from OAuth provider
- Email verification status from providers

### 7. Security Considerations
- Validate OAuth state parameter to prevent CSRF
- Use secure session cookies (httpOnly, secure, sameSite)
- Implement PKCE for additional security
- Validate redirect URIs
- Store tokens securely (encrypted in database if needed)
- Implement rate limiting on OAuth endpoints
- Use HTTPS in production
- Validate OAuth provider responses

### 8. File Structure
```
backend/
├── config/
│   ├── passport.ts
│   ├── redis.ts
│   └── oauth.ts
├── strategies/
│   ├── google.strategy.ts
│   ├── github.strategy.ts
│   └── facebook.strategy.ts
├── controllers/
│   └── auth.controller.ts
├── middleware/
│   ├── authenticate.ts
│   └── session.ts
├── models/
│   └── User.model.ts
├── routes/
│   └── auth.routes.ts
└── services/
    └── user.service.ts

frontend/
├── components/
│   ├── OAuthButtons.tsx
│   ├── AccountSettings.tsx
│   └── ProtectedRoute.tsx
├── pages/
│   ├── Login.tsx
│   └── Profile.tsx
└── hooks/
    └── useAuth.ts
```

### 9. Environment Variables
```
# Backend
NODE_ENV=production
SESSION_SECRET=your_session_secret
REDIS_URL=redis://localhost:6379
FRONTEND_URL=http://localhost:3000

# Google OAuth
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=http://localhost:5000/api/auth/google/callback

# GitHub OAuth
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
GITHUB_CALLBACK_URL=http://localhost:5000/api/auth/github/callback

# Facebook OAuth
FACEBOOK_APP_ID=your_facebook_app_id
FACEBOOK_APP_SECRET=your_facebook_app_secret
FACEBOOK_CALLBACK_URL=http://localhost:5000/api/auth/facebook/callback
```

### 10. OAuth Scopes Required
- **Google**: email, profile
- **GitHub**: user:email, read:user
- **Facebook**: email, public_profile

## Deliverables
- Complete OAuth implementation for all three providers
- Session management with Redis
- Account linking/unlinking functionality
- Frontend OAuth flow UI
- Comprehensive error handling
- Security best practices implemented
- Setup documentation for OAuth app registration

## Testing Checklist
- [ ] User can login with Google
- [ ] User can login with GitHub
- [ ] User can login with Facebook
- [ ] Same email across providers merges accounts
- [ ] User can link additional providers
- [ ] User can unlink providers (with validation)
- [ ] Session persists across requests
- [ ] Session expires after timeout
- [ ] Logout destroys session completely
- [ ] OAuth errors are handled gracefully
- [ ] State parameter prevents CSRF attacks
- [ ] Return URL works correctly after OAuth
- [ ] Profile data is fetched from providers
- [ ] Protected routes work correctly
