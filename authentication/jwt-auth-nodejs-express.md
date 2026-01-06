# JWT Authentication - Node.js & Express

## Objective
Implement a secure JWT-based authentication system with refresh tokens for a Node.js/Express backend API.

## Technology Stack
- Node.js 18+
- Express.js
- TypeScript
- jsonwebtoken library
- bcrypt for password hashing
- PostgreSQL or MongoDB
- express-validator for input validation

## Requirements

### 1. Database Schema
User table/collection with:
- id (UUID/ObjectId)
- email (unique, indexed)
- password (hashed)
- refreshToken (nullable)
- createdAt
- updatedAt
- isEmailVerified (boolean)
- role (enum: user, admin, etc.)

### 2. Authentication Endpoints
- **POST /api/auth/register** - User registration
- **POST /api/auth/login** - User login (returns access + refresh tokens)
- **POST /api/auth/refresh** - Refresh access token using refresh token
- **POST /api/auth/logout** - Invalidate refresh token
- **POST /api/auth/forgot-password** - Send password reset email
- **POST /api/auth/reset-password** - Reset password with token
- **GET /api/auth/verify-email/:token** - Verify email address
- **GET /api/auth/me** - Get current user (protected route)

### 3. Token Strategy
- **Access Token**: Short-lived (15 minutes), contains user ID and role
- **Refresh Token**: Long-lived (7 days), stored in database, httpOnly cookie
- Both tokens signed with different secrets
- Access token sent in response body
- Refresh token sent as httpOnly, secure cookie

### 4. Middleware
Create the following middleware:
- **authenticate**: Verify access token and attach user to request
- **authorize(roles)**: Check user role permissions
- **validateRequest**: Validate and sanitize input using express-validator
- **errorHandler**: Global error handling middleware
- **rateLimiter**: Rate limiting for auth endpoints

### 5. Security Features
- Password hashing with bcrypt (salt rounds: 12)
- JWT token signing with RS256 or HS256
- Refresh token rotation on use
- XSS protection headers
- CORS configuration
- Helmet.js for security headers
- Input sanitization
- SQL injection prevention
- Rate limiting on login attempts
- Account lockout after failed attempts

### 6. File Structure
```
src/
├── config/
│   ├── database.ts
│   └── jwt.ts
├── controllers/
│   └── auth.controller.ts
├── middleware/
│   ├── authenticate.ts
│   ├── authorize.ts
│   ├── validate.ts
│   └── errorHandler.ts
├── models/
│   └── User.model.ts
├── routes/
│   └── auth.routes.ts
├── services/
│   ├── auth.service.ts
│   ├── token.service.ts
│   └── email.service.ts
├── utils/
│   ├── AppError.ts
│   └── catchAsync.ts
├── validators/
│   └── auth.validator.ts
└── types/
    └── express.d.ts
```

### 7. Environment Variables
```
NODE_ENV=development
PORT=5000
DATABASE_URL=your_database_url
JWT_ACCESS_SECRET=your_access_secret
JWT_REFRESH_SECRET=your_refresh_secret
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
BCRYPT_ROUNDS=12
CORS_ORIGIN=http://localhost:3000
EMAIL_SERVICE_API_KEY=your_email_api_key
```

### 8. Response Format
```typescript
// Success
{
  success: true,
  data: {
    user: { id, email, role },
    accessToken: "jwt_token_here"
  }
}

// Error
{
  success: false,
  error: {
    message: "Error message",
    code: "ERROR_CODE",
    statusCode: 400
  }
}
```

### 9. Password Requirements
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

## Deliverables
- Complete authentication API with all endpoints
- Fully typed TypeScript implementation
- Database models and migrations
- Comprehensive error handling
- Security middleware
- Input validation on all endpoints
- API documentation (OpenAPI/Swagger)

## Testing Checklist
- [ ] User can register with valid credentials
- [ ] Registration fails with weak password
- [ ] Registration fails with duplicate email
- [ ] User can login with correct credentials
- [ ] Login fails with incorrect credentials
- [ ] Access token grants access to protected routes
- [ ] Expired access token is rejected
- [ ] Refresh token can generate new access token
- [ ] Logout invalidates refresh token
- [ ] Password reset email is sent
- [ ] Password can be reset with valid token
- [ ] Email verification works correctly
- [ ] Rate limiting prevents brute force attacks
- [ ] Tokens are properly signed and verified
