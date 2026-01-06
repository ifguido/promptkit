# GraphQL API - Apollo Server

## Objective
Build a production-ready GraphQL API using Apollo Server with TypeScript, featuring subscriptions, authentication, data loaders, and comprehensive error handling.

## Technology Stack
- Node.js 18+
- Apollo Server 4+
- TypeScript 5+
- GraphQL 16+
- PostgreSQL with Prisma
- DataLoader for batching
- graphql-shield for permissions
- Redis for subscriptions (PubSub)
- Jest for testing

## Requirements

### 1. Project Structure
```
src/
├── schema/
│   ├── index.ts
│   ├── typeDefs/
│   │   ├── user.graphql
│   │   ├── post.graphql
│   │   ├── comment.graphql
│   │   └── scalars.graphql
│   └── resolvers/
│       ├── index.ts
│       ├── user.resolver.ts
│       ├── post.resolver.ts
│       ├── comment.resolver.ts
│       ├── subscription.resolver.ts
│       └── scalars.resolver.ts
├── models/
│   ├── User.model.ts
│   ├── Post.model.ts
│   └── Comment.model.ts
├── services/
│   ├── user.service.ts
│   ├── post.service.ts
│   └── comment.service.ts
├── dataloaders/
│   ├── index.ts
│   ├── user.loader.ts
│   └── post.loader.ts
├── middleware/
│   ├── auth.middleware.ts
│   ├── permissions.ts
│   └── errorHandler.ts
├── directives/
│   ├── auth.directive.ts
│   ├── rateLimit.directive.ts
│   └── deprecated.directive.ts
├── utils/
│   ├── context.ts
│   ├── errors.ts
│   └── validation.ts
├── types/
│   └── context.types.ts
├── __tests__/
│   ├── integration/
│   └── unit/
├── config/
│   ├── apollo.ts
│   └── database.ts
└── server.ts
```

### 2. GraphQL Schema

**Type Definitions:**
```graphql
# User Types
type User {
  id: ID!
  email: String!
  firstName: String!
  lastName: String!
  fullName: String! # Computed field
  avatar: String
  role: Role!
  posts(
    first: Int
    after: String
    orderBy: PostOrderBy
  ): PostConnection!
  createdAt: DateTime!
  updatedAt: DateTime!
}

enum Role {
  USER
  EDITOR
  ADMIN
}

# Post Types
type Post {
  id: ID!
  title: String!
  content: String!
  excerpt: String # Computed field
  published: Boolean!
  author: User!
  comments: [Comment!]!
  tags: [String!]!
  viewCount: Int!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# Comment Types
type Comment {
  id: ID!
  content: String!
  author: User!
  post: Post!
  createdAt: DateTime!
  updatedAt: DateTime!
}

# Queries
type Query {
  # User queries
  me: User @auth
  user(id: ID!): User
  users(
    first: Int
    after: String
    filter: UserFilter
    orderBy: UserOrderBy
  ): UserConnection!

  # Post queries
  post(id: ID!): Post
  posts(
    first: Int = 10
    after: String
    filter: PostFilter
    orderBy: PostOrderBy
  ): PostConnection!
  searchPosts(query: String!): [Post!]!

  # Comment queries
  comment(id: ID!): Comment
  comments(postId: ID!): [Comment!]!
}

# Mutations
type Mutation {
  # Auth mutations
  register(input: RegisterInput!): AuthPayload!
  login(input: LoginInput!): AuthPayload!

  # User mutations
  updateProfile(input: UpdateProfileInput!): User! @auth
  changePassword(input: ChangePasswordInput!): Boolean! @auth

  # Post mutations
  createPost(input: CreatePostInput!): Post! @auth
  updatePost(id: ID!, input: UpdatePostInput!): Post! @auth
  deletePost(id: ID!): Boolean! @auth
  publishPost(id: ID!): Post! @auth

  # Comment mutations
  createComment(input: CreateCommentInput!): Comment! @auth
  updateComment(id: ID!, input: UpdateCommentInput!): Comment! @auth
  deleteComment(id: ID!): Boolean! @auth
}

# Subscriptions
type Subscription {
  postCreated: Post!
  postUpdated(id: ID!): Post!
  commentAdded(postId: ID!): Comment!
}

# Input Types
input RegisterInput {
  email: String!
  password: String!
  firstName: String!
  lastName: String!
}

input LoginInput {
  email: String!
  password: String!
}

input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
  published: Boolean = false
}

input UpdatePostInput {
  title: String
  content: String
  tags: [String!]
  published: Boolean
}

input CreateCommentInput {
  postId: ID!
  content: String!
}

# Custom Scalars
scalar DateTime
scalar Email
scalar URL

# Auth Payload
type AuthPayload {
  token: String!
  user: User!
}

# Filters
input PostFilter {
  published: Boolean
  authorId: ID
  tags: [String!]
  search: String
}

input UserFilter {
  role: Role
  isActive: Boolean
}

# Ordering
enum PostOrderBy {
  CREATED_AT_ASC
  CREATED_AT_DESC
  TITLE_ASC
  TITLE_DESC
  VIEW_COUNT_DESC
}

enum UserOrderBy {
  CREATED_AT_ASC
  CREATED_AT_DESC
  EMAIL_ASC
  EMAIL_DESC
}
```

### 3. Resolvers Implementation

**User Resolver:**
```typescript
export const userResolvers = {
  Query: {
    me: async (_: any, __: any, context: Context) => {
      return context.user;
    },
    user: async (_: any, { id }: { id: string }, context: Context) => {
      return context.loaders.userLoader.load(id);
    },
    users: async (_: any, args: PaginationArgs, context: Context) => {
      return userService.findAll(args, context);
    },
  },

  User: {
    fullName: (parent: User) => {
      return `${parent.firstName} ${parent.lastName}`;
    },
    posts: async (parent: User, args: PaginationArgs, context: Context) => {
      return context.loaders.userPostsLoader.load({
        userId: parent.id,
        ...args
      });
    },
  },

  Mutation: {
    updateProfile: async (_: any, { input }: any, context: Context) => {
      return userService.update(context.user.id, input);
    },
  },
};
```

### 4. DataLoader Implementation

**User Loader:**
```typescript
export const createUserLoader = (prisma: PrismaClient) => {
  return new DataLoader<string, User>(async (ids) => {
    const users = await prisma.user.findMany({
      where: { id: { in: [...ids] } },
    });

    const userMap = new Map(users.map(user => [user.id, user]));
    return ids.map(id => userMap.get(id) || null);
  });
};
```

### 5. Authentication & Authorization

**Auth Directive:**
```typescript
@Directive('@auth')
class AuthDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field: GraphQLField<any, any>) {
    const { resolve = defaultFieldResolver } = field;

    field.resolve = async function (...args) {
      const context = args[2];
      if (!context.user) {
        throw new AuthenticationError('Not authenticated');
      }
      return resolve.apply(this, args);
    };
  }
}
```

**Permission Layer (graphql-shield):**
```typescript
const permissions = shield({
  Query: {
    me: isAuthenticated,
    users: isAdmin,
  },
  Mutation: {
    createPost: isAuthenticated,
    updatePost: and(isAuthenticated, isPostOwner),
    deletePost: and(isAuthenticated, or(isPostOwner, isAdmin)),
    publishPost: and(isAuthenticated, hasRole('EDITOR')),
  },
});
```

### 6. Subscriptions

**Post Subscription:**
```typescript
export const subscriptionResolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator(['POST_CREATED']),
    },
    commentAdded: {
      subscribe: (_: any, { postId }: { postId: string }) => {
        return pubsub.asyncIterator([`COMMENT_ADDED_${postId}`]);
      },
    },
  },
};

// Trigger subscription
await pubsub.publish('POST_CREATED', { postCreated: newPost });
```

### 7. Context Builder

```typescript
interface Context {
  user: User | null;
  prisma: PrismaClient;
  loaders: {
    userLoader: DataLoader<string, User>;
    postLoader: DataLoader<string, Post>;
    userPostsLoader: DataLoader<any, Post[]>;
  };
  pubsub: PubSub;
}

const context = async ({ req }: { req: Request }): Promise<Context> => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const user = token ? await verifyToken(token) : null;

  return {
    user,
    prisma,
    loaders: {
      userLoader: createUserLoader(prisma),
      postLoader: createPostLoader(prisma),
      userPostsLoader: createUserPostsLoader(prisma),
    },
    pubsub,
  };
};
```

### 8. Error Handling

```typescript
export class AppError extends ApolloError {
  constructor(message: string, code: string) {
    super(message, code);
  }
}

export class ValidationError extends AppError {
  constructor(message: string, fields?: any) {
    super(message, 'VALIDATION_ERROR');
    this.extensions = { fields };
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 'NOT_FOUND');
  }
}
```

### 9. Features

**Performance:**
- DataLoader for N+1 query prevention
- Query complexity analysis
- Depth limiting
- Field-level caching
- Persisted queries

**Security:**
- Query depth limiting
- Query complexity calculation
- Rate limiting per operation
- Input validation
- SQL injection prevention

**Developer Experience:**
- GraphQL Playground in development
- Auto-generated TypeScript types (graphql-codegen)
- Schema-first or code-first approach
- Comprehensive error messages

**Monitoring:**
- Apollo Studio integration
- Performance tracing
- Error tracking
- Query analytics

### 10. Environment Variables
```
NODE_ENV=development
PORT=4000
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
REDIS_URL=redis://localhost:6379
JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN=1h
APOLLO_KEY=your_apollo_key
APOLLO_GRAPH_REF=your_graph_ref
MAX_QUERY_DEPTH=7
MAX_QUERY_COMPLEXITY=1000
```

## Deliverables
- Complete GraphQL API with all features
- Type-safe schema and resolvers
- Authentication & authorization system
- DataLoader implementation
- Subscription support
- Comprehensive error handling
- Unit and integration tests
- Apollo Studio integration
- Auto-generated TypeScript types
- Documentation

## Testing Checklist
- [ ] All queries return correct data
- [ ] All mutations modify data correctly
- [ ] Subscriptions emit events properly
- [ ] Authentication blocks unauthenticated users
- [ ] Authorization enforces permissions
- [ ] DataLoaders prevent N+1 queries
- [ ] Pagination works correctly
- [ ] Filtering and sorting work
- [ ] Error handling provides useful messages
- [ ] Query depth limiting prevents abuse
- [ ] Complexity analysis blocks expensive queries
- [ ] Custom scalars validate correctly
- [ ] Directives apply correctly
- [ ] Tests achieve >80% coverage
- [ ] Performance is acceptable
