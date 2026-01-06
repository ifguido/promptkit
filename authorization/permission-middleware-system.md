# Permission-Based Authorization Middleware

## Objective
Build a flexible, composable permission middleware system for Express.js that supports complex authorization rules, resource ownership, and dynamic permission evaluation.

## Technology Stack
- Node.js/Express with TypeScript
- Database-agnostic (works with any ORM/DB)
- Redis for permission caching
- Express middleware pattern

## Requirements

### 1. Core Middleware Functions

**authorize(permission)**
```typescript
// Simple permission check
app.get('/posts', authorize('posts.read'), controller);

// Multiple permissions (AND logic)
app.post('/posts', authorize(['posts.create', 'media.upload']), controller);

// Multiple permissions (OR logic)
app.get('/admin', authorize.any(['admin.access', 'superadmin.access']), controller);
```

**authorizeOwner(resourceType, ownerField)**
```typescript
// Check if user owns the resource
app.put('/posts/:id', authorizeOwner('post', 'userId'), controller);

// Custom ownership check
app.delete('/comments/:id', authorizeOwner('comment', async (req, resource) => {
  return resource.userId === req.user.id || resource.post.userId === req.user.id;
}), controller);
```

**authorizeRole(...roles)**
```typescript
// Role-based access
app.get('/admin/dashboard', authorizeRole('admin', 'moderator'), controller);
```

**authorizeCondition(conditionFn)**
```typescript
// Dynamic conditional authorization
app.post('/posts/:id/publish', authorizeCondition(async (req) => {
  const post = await Post.findById(req.params.id);
  return post.status === 'draft' && post.userId === req.user.id;
}), controller);
```

### 2. Permission Builder Pattern
```typescript
const { permit } = require('./middleware/authorization');

// Fluent API for complex rules
app.put('/posts/:id',
  permit('posts.update')
    .or('posts.update.any')
    .andOwns('post')
    .unless((req) => req.resource.published && !req.user.hasRole('admin'))
    .withCache(300) // Cache for 5 minutes
    .check(),
  controller
);
```

### 3. Resource Loading & Context
Automatically load and attach resources to request:
```typescript
// Middleware loads resource and attaches to req.resource
app.use('/posts/:id', loadResource('post', 'id'));

// Now authorization middleware can access req.resource
app.put('/posts/:id',
  authorizeOwner('post', 'userId'),
  controller
);
```

### 4. Permission Policy Classes
```typescript
// Define reusable authorization policies
class PostPolicy {
  static canCreate(user: User): boolean {
    return user.hasPermission('posts.create');
  }

  static canUpdate(user: User, post: Post): boolean {
    return user.id === post.userId || user.hasPermission('posts.update.any');
  }

  static canDelete(user: User, post: Post): boolean {
    return (user.id === post.userId && post.status === 'draft') ||
           user.hasRole('admin');
  }

  static canPublish(user: User, post: Post): boolean {
    return post.status === 'draft' &&
           (user.id === post.userId || user.hasRole('editor'));
  }
}

// Use in routes
app.put('/posts/:id', authorizePolicy(PostPolicy, 'canUpdate'), controller);
```

### 5. Scope-Based Authorization
```typescript
// Automatically filter queries based on user permissions
app.get('/posts',
  applyScope('posts', (query, user) => {
    if (!user.hasPermission('posts.read.any')) {
      query.where('userId', user.id);
    }
    if (user.hasRole('viewer')) {
      query.where('status', 'published');
    }
    return query;
  }),
  controller
);
```

### 6. File Structure
```
src/
├── middleware/
│   ├── authorization/
│   │   ├── index.ts
│   │   ├── authorize.ts
│   │   ├── authorizeOwner.ts
│   │   ├── authorizeRole.ts
│   │   ├── authorizeCondition.ts
│   │   ├── authorizePolicy.ts
│   │   ├── loadResource.ts
│   │   ├── applyScope.ts
│   │   └── PermissionBuilder.ts
│   └── authenticate.ts
├── policies/
│   ├── PostPolicy.ts
│   ├── UserPolicy.ts
│   └── CommentPolicy.ts
├── services/
│   ├── PermissionService.ts
│   └── CacheService.ts
└── types/
    ├── authorization.types.ts
    └── express.d.ts
```

### 7. Error Handling
Consistent authorization error responses:
```typescript
// 401 - Not authenticated
{ error: 'Authentication required', code: 'AUTH_REQUIRED' }

// 403 - Not authorized
{
  error: 'Insufficient permissions',
  code: 'FORBIDDEN',
  required: ['posts.update'],
  context: { resourceId: '123', reason: 'not_owner' }
}

// 404 - Resource not found (avoid leaking existence)
{ error: 'Resource not found', code: 'NOT_FOUND' }
```

### 8. Features

**Performance:**
- Redis caching for permission checks
- Batch permission loading
- Lazy evaluation of conditions
- Query optimization for scope filters

**Flexibility:**
- Composable middleware
- Custom authorization logic
- Policy-based architecture
- Dynamic permission evaluation

**Debugging:**
- Detailed authorization logs
- Permission audit trail
- Dry-run mode to test permissions
- Authorization decision explanations

**Testing:**
- Mock permission checks
- Test helpers for authorization
- Permission fixtures

### 9. Advanced Patterns

**Time-Based Permissions:**
```typescript
authorize.during('posts.update', {
  after: '9:00',
  before: '17:00',
  timezone: 'America/New_York'
})
```

**Rate-Limited Permissions:**
```typescript
authorize('posts.create').rateLimit({
  max: 10,
  window: '1h',
  key: (req) => req.user.id
})
```

**Hierarchical Resources:**
```typescript
// Check parent resource ownership
authorize.owns('project').throughParent('task', 'projectId')
```

**Field-Level Permissions:**
```typescript
// Only allow updating specific fields
authorize('posts.update').fields(['title', 'content'])
  .unless('posts.update.any').fields('all')
```

## Deliverables
- Complete middleware suite with all authorization patterns
- Policy class examples
- Permission caching layer
- Comprehensive TypeScript types
- Unit tests for all middleware
- Integration test examples
- Documentation with usage examples

## Testing Checklist
- [ ] Simple permission checks work correctly
- [ ] Multiple permission checks (AND/OR) work
- [ ] Owner-based authorization validates correctly
- [ ] Role-based checks function properly
- [ ] Conditional authorization evaluates correctly
- [ ] Policy classes enforce rules properly
- [ ] Resource loading middleware works
- [ ] Scope-based filtering applies correctly
- [ ] Permission caching improves performance
- [ ] Error messages are clear and consistent
- [ ] Audit logs capture all decisions
- [ ] Complex permission chains work
- [ ] Time-based permissions activate/deactivate correctly
- [ ] Rate limiting enforces limits
- [ ] Field-level permissions restrict correctly
