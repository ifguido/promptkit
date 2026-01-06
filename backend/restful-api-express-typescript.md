# RESTful API - Express & TypeScript

## Objective
Build a production-ready RESTful API using Express.js and TypeScript with best practices, proper error handling, validation, documentation, and testing.

## Technology Stack
- Node.js 18+
- Express.js 4+
- TypeScript 5+
- PostgreSQL or MongoDB
- Prisma or TypeORM
- express-validator or Zod
- Jest for testing
- Swagger/OpenAPI for documentation

## Requirements

### 1. Project Structure
```
src/
├── config/
│   ├── database.ts
│   ├── environment.ts
│   └── swagger.ts
├── controllers/
│   ├── auth.controller.ts
│   ├── user.controller.ts
│   └── post.controller.ts
├── middleware/
│   ├── authenticate.ts
│   ├── authorize.ts
│   ├── validate.ts
│   ├── errorHandler.ts
│   ├── requestLogger.ts
│   └── rateLimiter.ts
├── models/
│   ├── User.model.ts
│   └── Post.model.ts
├── routes/
│   ├── index.ts
│   ├── auth.routes.ts
│   ├── user.routes.ts
│   └── post.routes.ts
├── services/
│   ├── auth.service.ts
│   ├── user.service.ts
│   └── post.service.ts
├── validators/
│   ├── auth.validator.ts
│   ├── user.validator.ts
│   └── post.validator.ts
├── utils/
│   ├── ApiError.ts
│   ├── ApiResponse.ts
│   ├── catchAsync.ts
│   └── pagination.ts
├── types/
│   ├── express.d.ts
│   └── api.types.ts
├── __tests__/
│   ├── integration/
│   └── unit/
├── app.ts
└── server.ts
```

### 2. Core API Endpoints

**Authentication:**
- POST /api/v1/auth/register
- POST /api/v1/auth/login
- POST /api/v1/auth/logout
- POST /api/v1/auth/refresh-token
- POST /api/v1/auth/forgot-password
- POST /api/v1/auth/reset-password

**Users:**
- GET /api/v1/users (paginated, filterable, sortable)
- GET /api/v1/users/:id
- POST /api/v1/users (admin only)
- PATCH /api/v1/users/:id
- DELETE /api/v1/users/:id
- GET /api/v1/users/me

**Posts (example resource):**
- GET /api/v1/posts (paginated, filtered, sorted)
- GET /api/v1/posts/:id
- POST /api/v1/posts
- PATCH /api/v1/posts/:id
- DELETE /api/v1/posts/:id
- POST /api/v1/posts/:id/publish
- GET /api/v1/posts/:id/comments

### 3. Response Format

**Success Response:**
```typescript
{
  success: true,
  data: {
    // Resource data
  },
  message: "Operation successful",
  metadata: {
    timestamp: "2024-01-01T00:00:00Z",
    requestId: "uuid"
  }
}
```

**Error Response:**
```typescript
{
  success: false,
  error: {
    code: "VALIDATION_ERROR",
    message: "Validation failed",
    details: [
      {
        field: "email",
        message: "Invalid email format"
      }
    ]
  },
  metadata: {
    timestamp: "2024-01-01T00:00:00Z",
    requestId: "uuid"
  }
}
```

**Paginated Response:**
```typescript
{
  success: true,
  data: [...],
  pagination: {
    page: 1,
    limit: 20,
    totalPages: 5,
    totalItems: 100,
    hasNext: true,
    hasPrev: false
  }
}
```

### 4. Features

**Request Validation:**
- Input validation with express-validator or Zod
- Request body, params, and query validation
- Sanitization to prevent XSS
- Custom validation rules

**Error Handling:**
- Centralized error handling middleware
- Custom error classes
- Proper HTTP status codes
- Development vs production error details
- Error logging

**Security:**
- Helmet.js for security headers
- CORS configuration
- Rate limiting (express-rate-limit)
- Request size limits
- SQL injection prevention
- XSS protection
- CSRF tokens for non-API routes

**Logging:**
- Request/response logging
- Error logging
- Performance metrics
- Winston or Pino logger
- Log rotation
- Different log levels per environment

**Database:**
- Connection pooling
- Transactions support
- Soft deletes
- Timestamps (createdAt, updatedAt)
- Query optimization
- Database migrations

**Performance:**
- Compression middleware
- Response caching (Redis)
- Database query optimization
- Pagination for large datasets
- ETag support

### 5. Middleware Stack

**Global Middleware:**
```typescript
app.use(helmet());
app.use(cors(corsOptions));
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));
app.use(compression());
app.use(requestLogger);
app.use(rateLimiter);
```

**Route-Specific:**
```typescript
router.post(
  '/posts',
  authenticate,
  authorize('posts.create'),
  validate(createPostSchema),
  postController.create
);
```

### 6. Service Layer Pattern

**Example Service:**
```typescript
class UserService {
  async findAll(filters: UserFilters, pagination: Pagination) {
    // Business logic
  }

  async findById(id: string) {
    // Business logic
  }

  async create(data: CreateUserDto) {
    // Business logic
  }

  async update(id: string, data: UpdateUserDto) {
    // Business logic
  }

  async delete(id: string) {
    // Business logic
  }
}
```

### 7. Controller Pattern

**Example Controller:**
```typescript
export const getUsers = catchAsync(async (req, res) => {
  const { page = 1, limit = 10, sort, filter } = req.query;

  const result = await userService.findAll(
    parseFilters(filter),
    { page: Number(page), limit: Number(limit), sort }
  );

  res.status(200).json(
    new ApiResponse(result.data, 'Users retrieved successfully', {
      pagination: result.pagination
    })
  );
});
```

### 8. Database Models (Prisma Example)

```prisma
model User {
  id            String   @id @default(uuid())
  email         String   @unique
  password      String
  firstName     String
  lastName      String
  role          Role     @default(USER)
  isActive      Boolean  @default(true)
  emailVerified Boolean  @default(false)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  posts         Post[]
}

model Post {
  id          String   @id @default(uuid())
  title       String
  content     String
  published   Boolean  @default(false)
  authorId    String
  author      User     @relation(fields: [authorId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

### 9. Environment Variables
```
NODE_ENV=development
PORT=5000
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
REDIS_URL=redis://localhost:6379
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN=1h
CORS_ORIGIN=http://localhost:3000
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
LOG_LEVEL=info
```

### 10. API Documentation
- Swagger/OpenAPI specification
- Auto-generated from code annotations
- Interactive API explorer
- Request/response examples
- Authentication documentation

### 11. Testing

**Unit Tests:**
- Service layer tests
- Utility function tests
- Validation schema tests

**Integration Tests:**
- API endpoint tests
- Database integration tests
- Authentication flow tests

**Test Coverage:**
- Minimum 80% code coverage
- Critical paths 100% covered

## Deliverables
- Complete REST API implementation
- Comprehensive error handling
- Input validation on all endpoints
- Authentication & authorization
- API documentation (Swagger)
- Database models and migrations
- Unit and integration tests
- Docker setup
- README with setup instructions

## Testing Checklist
- [ ] All endpoints return correct status codes
- [ ] Validation errors are descriptive
- [ ] Authentication is enforced on protected routes
- [ ] Authorization checks user permissions
- [ ] Pagination works correctly
- [ ] Filtering and sorting work
- [ ] Error handling catches all errors
- [ ] Rate limiting prevents abuse
- [ ] CORS is configured correctly
- [ ] Database transactions work
- [ ] Soft deletes function properly
- [ ] Logging captures important events
- [ ] API documentation is accurate
- [ ] Tests achieve >80% coverage
- [ ] Performance is acceptable under load
