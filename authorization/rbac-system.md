# Role-Based Access Control (RBAC) System

## Objective
Implement a comprehensive Role-Based Access Control system with hierarchical roles, permissions, and resource-level access control.

## Technology Stack
- Backend: Node.js/Express with TypeScript
- Database: PostgreSQL with proper relations
- Frontend: React with TypeScript
- State Management: React Context or Redux

## Requirements

### 1. Database Schema

**Roles Table:**
- id (UUID)
- name (unique: admin, manager, editor, viewer)
- description
- priority (number for hierarchy)
- createdAt, updatedAt

**Permissions Table:**
- id (UUID)
- resource (e.g., 'users', 'posts', 'settings')
- action (e.g., 'create', 'read', 'update', 'delete')
- description
- unique constraint on (resource, action)

**Role_Permissions Table:**
- roleId (FK to Roles)
- permissionId (FK to Permissions)
- primary key (roleId, permissionId)

**Users Table:**
- id (UUID)
- email
- roleId (FK to Roles)
- customPermissions (JSON - for exceptions)
- isActive (boolean)

### 2. Role Hierarchy
Define the following roles with inheritance:
- **admin**: Full access to everything
- **manager**: Can manage users and content, cannot change system settings
- **editor**: Can create and edit content, cannot delete or manage users
- **viewer**: Read-only access to content

### 3. Backend Implementation

**API Endpoints:**
- **GET /api/roles** - List all roles
- **GET /api/roles/:id/permissions** - Get permissions for a role
- **POST /api/roles** - Create new role (admin only)
- **PUT /api/roles/:id** - Update role (admin only)
- **DELETE /api/roles/:id** - Delete role (admin only)
- **POST /api/roles/:id/permissions** - Add permissions to role
- **DELETE /api/roles/:id/permissions/:permissionId** - Remove permission
- **GET /api/permissions** - List all available permissions
- **PUT /api/users/:id/role** - Assign role to user
- **POST /api/users/:id/permissions** - Grant custom permissions to user
- **GET /api/users/:id/permissions** - Get all effective permissions for user

**Middleware:**
- **authorize(resource, action)**: Check if user has permission
- **requireRole(...roles)**: Check if user has one of the required roles
- **checkResourceOwnership(resource)**: Verify user owns the resource

### 4. Permission Checking Logic
Implement smart permission resolution:
1. Check if user has custom permission grant
2. Check if user has custom permission denial
3. Check role-based permissions
4. Apply role hierarchy (higher roles inherit lower role permissions)
5. Check resource ownership for user-scoped resources

### 5. Frontend Components

Create the following:
- **RoleManagement Component** - Admin UI to manage roles
- **PermissionMatrix Component** - Visual grid showing role-permission mappings
- **UserRoleAssignment Component** - Assign roles to users
- **ProtectedComponent Wrapper** - Show/hide UI based on permissions
- **PermissionGate Component** - Conditional rendering based on permissions

### 6. Frontend Hooks
```typescript
// Usage examples
const { hasPermission } = usePermissions();
const canEdit = hasPermission('posts', 'update');

const { hasRole } = useAuth();
const isAdmin = hasRole('admin');

const { can } = useAbility();
const canDeletePost = can('delete', 'posts', postId);
```

### 7. File Structure
```
backend/
├── models/
│   ├── Role.model.ts
│   ├── Permission.model.ts
│   └── User.model.ts
├── middleware/
│   ├── authorize.ts
│   ├── requireRole.ts
│   └── checkOwnership.ts
├── services/
│   ├── rbac.service.ts
│   └── permission.service.ts
├── controllers/
│   ├── role.controller.ts
│   └── permission.controller.ts
└── routes/
    ├── role.routes.ts
    └── permission.routes.ts

frontend/
├── components/
│   ├── rbac/
│   │   ├── RoleManagement.tsx
│   │   ├── PermissionMatrix.tsx
│   │   └── UserRoleAssignment.tsx
│   └── ProtectedComponent.tsx
├── hooks/
│   ├── usePermissions.ts
│   ├── useAbility.ts
│   └── useRole.ts
└── contexts/
    └── PermissionContext.tsx
```

### 8. Permission Definitions
Define comprehensive permissions for common resources:
```typescript
const PERMISSIONS = {
  users: ['create', 'read', 'update', 'delete', 'assign_role'],
  posts: ['create', 'read', 'update', 'delete', 'publish'],
  comments: ['create', 'read', 'update', 'delete', 'moderate'],
  settings: ['read', 'update'],
  analytics: ['read'],
  audit_logs: ['read'],
};
```

### 9. Advanced Features
- **Temporary Permission Grants**: Time-limited permissions
- **Resource-Level Permissions**: Per-item access control
- **Permission Inheritance**: Child resources inherit parent permissions
- **Audit Logging**: Track all permission checks and changes
- **Cached Permission Checks**: Redis caching for performance
- **Bulk Permission Updates**: Update multiple users/roles at once

### 10. Security Considerations
- Validate all permission checks on backend (never trust frontend)
- Implement principle of least privilege
- Audit all role/permission changes
- Prevent privilege escalation
- Validate role hierarchy to prevent cycles
- Ensure admins can't lock themselves out

## Deliverables
- Complete RBAC system with database schema
- Backend middleware for authorization
- Admin UI for role/permission management
- Frontend permission checking utilities
- Seeder script for initial roles and permissions
- Comprehensive API documentation
- Migration scripts

## Testing Checklist
- [ ] Admin can create/update/delete roles
- [ ] Admin can assign permissions to roles
- [ ] Users inherit permissions from their role
- [ ] Custom user permissions override role permissions
- [ ] Role hierarchy works correctly
- [ ] Permission checks prevent unauthorized access
- [ ] Resource ownership is validated correctly
- [ ] Permission changes take effect immediately
- [ ] Audit logs capture all authorization events
- [ ] Frontend UI reflects user permissions
- [ ] Cannot escalate own privileges
- [ ] Cannot assign higher role than own role
- [ ] Cached permissions invalidate correctly
- [ ] Bulk operations work as expected
