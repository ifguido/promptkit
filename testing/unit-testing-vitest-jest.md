# Unit Testing - Vitest & Jest

## Objective
Implement comprehensive unit testing for a full-stack TypeScript application using Vitest (or Jest), with focus on high coverage, mocking strategies, and testing best practices.

## Technology Stack
- Vitest (or Jest as alternative)
- TypeScript
- React Testing Library (for React components)
- MSW (Mock Service Worker) for API mocking
- @testing-library/jest-dom for custom matchers
- @testing-library/user-event for user interactions

## Requirements

### 1. Test Coverage Areas

**Frontend Testing:**
- React components (presentational & container)
- Custom hooks
- Context providers
- Utility functions
- Form validation logic
- State management (Redux, Zustand, etc.)

**Backend Testing:**
- Controllers
- Services/business logic
- Middleware
- Utility functions
- Database models
- API validators

**Integration Testing:**
- API endpoints
- Database operations
- Authentication flows

### 2. Project Structure
```
tests/
├── unit/
│   ├── components/
│   │   ├── Button.test.tsx
│   │   ├── Form.test.tsx
│   │   └── Modal.test.tsx
│   ├── hooks/
│   │   ├── useAuth.test.ts
│   │   ├── useFetch.test.ts
│   │   └── useLocalStorage.test.ts
│   ├── utils/
│   │   ├── validation.test.ts
│   │   ├── formatters.test.ts
│   │   └── helpers.test.ts
│   └── services/
│       ├── auth.service.test.ts
│       └── user.service.test.ts
├── integration/
│   ├── api/
│   │   ├── auth.api.test.ts
│   │   └── users.api.test.ts
│   └── database/
│       └── user.repository.test.ts
├── mocks/
│   ├── handlers.ts
│   ├── server.ts
│   └── data.ts
├── fixtures/
│   ├── user.fixture.ts
│   └── product.fixture.ts
├── setup/
│   ├── setup.ts
│   └── teardown.ts
└── vitest.config.ts
```

### 3. Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup/setup.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.ts',
        '**/types/',
        '**/*.stories.tsx',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
    mockReset: true,
    restoreMocks: true,
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### 4. Test Setup

**Setup File:**
```typescript
// tests/setup/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';
import { server } from '../mocks/server';

// Start MSW server
beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterAll(() => server.close());
afterEach(() => {
  server.resetHandlers();
  cleanup();
});

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  takeRecords() {
    return [];
  }
  unobserve() {}
};
```

### 5. Component Testing

**Simple Component Test:**
```typescript
// components/Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  it('calls onClick handler when clicked', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    const user = userEvent.setup();
    await user.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('applies custom className', () => {
    render(<Button className="custom-class">Click me</Button>);
    expect(screen.getByRole('button')).toHaveClass('custom-class');
  });
});
```

**Form Component Test:**
```typescript
// components/LoginForm.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('submits form with valid data', async () => {
    const onSubmit = vi.fn();
    render(<LoginForm onSubmit={onSubmit} />);

    const user = userEvent.setup();

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /login/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        password: 'password123',
      });
    });
  });

  it('shows validation errors for invalid data', async () => {
    render(<LoginForm onSubmit={vi.fn()} />);

    const user = userEvent.setup();

    await user.type(screen.getByLabelText(/email/i), 'invalid-email');
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(await screen.findByText(/invalid email/i)).toBeInTheDocument();
  });

  it('disables submit button while loading', async () => {
    const onSubmit = vi.fn(() => new Promise((resolve) => setTimeout(resolve, 1000)));
    render(<LoginForm onSubmit={onSubmit} />);

    const user = userEvent.setup();
    const submitButton = screen.getByRole('button', { name: /login/i });

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(submitButton);

    expect(submitButton).toBeDisabled();
  });
});
```

### 6. Hook Testing

```typescript
// hooks/useAuth.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { useAuth } from './useAuth';
import { AuthProvider } from '../contexts/AuthContext';

const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>;

describe('useAuth', () => {
  it('returns null user when not authenticated', () => {
    const { result } = renderHook(() => useAuth(), { wrapper });
    expect(result.current.user).toBeNull();
  });

  it('logs in user successfully', async () => {
    const { result } = renderHook(() => useAuth(), { wrapper });

    await act(async () => {
      await result.current.login('test@example.com', 'password123');
    });

    await waitFor(() => {
      expect(result.current.user).not.toBeNull();
      expect(result.current.user?.email).toBe('test@example.com');
    });
  });

  it('logs out user successfully', async () => {
    const { result } = renderHook(() => useAuth(), { wrapper });

    // Login first
    await act(async () => {
      await result.current.login('test@example.com', 'password123');
    });

    // Then logout
    await act(async () => {
      await result.current.logout();
    });

    expect(result.current.user).toBeNull();
  });
});
```

### 7. Service/Business Logic Testing

```typescript
// services/user.service.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserService } from './user.service';
import { prisma } from '../lib/prisma';

vi.mock('../lib/prisma', () => ({
  prisma: {
    user: {
      findUnique: vi.fn(),
      create: vi.fn(),
      update: vi.fn(),
      delete: vi.fn(),
    },
  },
}));

describe('UserService', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  describe('findById', () => {
    it('returns user when found', async () => {
      const mockUser = { id: '1', email: 'test@example.com', name: 'Test User' };
      vi.mocked(prisma.user.findUnique).mockResolvedValue(mockUser);

      const result = await UserService.findById('1');

      expect(result).toEqual(mockUser);
      expect(prisma.user.findUnique).toHaveBeenCalledWith({
        where: { id: '1' },
      });
    });

    it('throws error when user not found', async () => {
      vi.mocked(prisma.user.findUnique).mockResolvedValue(null);

      await expect(UserService.findById('999')).rejects.toThrow('User not found');
    });
  });

  describe('create', () => {
    it('creates user with hashed password', async () => {
      const userData = {
        email: 'new@example.com',
        password: 'password123',
        name: 'New User',
      };

      const mockCreatedUser = { id: '2', ...userData, password: 'hashed' };
      vi.mocked(prisma.user.create).mockResolvedValue(mockCreatedUser);

      const result = await UserService.create(userData);

      expect(result.password).not.toBe('password123');
      expect(prisma.user.create).toHaveBeenCalled();
    });
  });
});
```

### 8. API Mocking with MSW

**MSW Handlers:**
```typescript
// tests/mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
  // Auth endpoints
  rest.post('/api/auth/login', async (req, res, ctx) => {
    const { email, password } = await req.json();

    if (email === 'test@example.com' && password === 'password123') {
      return res(
        ctx.status(200),
        ctx.json({
          user: { id: '1', email, name: 'Test User' },
          token: 'fake-jwt-token',
        })
      );
    }

    return res(
      ctx.status(401),
      ctx.json({ error: 'Invalid credentials' })
    );
  }),

  // User endpoints
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;

    return res(
      ctx.status(200),
      ctx.json({
        id,
        email: 'user@example.com',
        name: 'Test User',
      })
    );
  }),

  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(200),
      ctx.json({
        data: [
          { id: '1', email: 'user1@example.com', name: 'User 1' },
          { id: '2', email: 'user2@example.com', name: 'User 2' },
        ],
        pagination: { page: 1, totalPages: 1 },
      })
    );
  }),
];
```

**MSW Server:**
```typescript
// tests/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

### 9. Utility Function Testing

```typescript
// utils/validation.test.ts
import { describe, it, expect } from 'vitest';
import { validateEmail, validatePassword, validatePhone } from './validation';

describe('Validation Utilities', () => {
  describe('validateEmail', () => {
    it('returns true for valid emails', () => {
      expect(validateEmail('test@example.com')).toBe(true);
      expect(validateEmail('user+tag@domain.co.uk')).toBe(true);
    });

    it('returns false for invalid emails', () => {
      expect(validateEmail('invalid')).toBe(false);
      expect(validateEmail('@example.com')).toBe(false);
      expect(validateEmail('test@')).toBe(false);
    });
  });

  describe('validatePassword', () => {
    it('returns true for strong passwords', () => {
      expect(validatePassword('Password123!')).toBe(true);
    });

    it('returns false for weak passwords', () => {
      expect(validatePassword('weak')).toBe(false);
      expect(validatePassword('12345678')).toBe(false);
      expect(validatePassword('password')).toBe(false);
    });
  });
});
```

### 10. Snapshot Testing

```typescript
// components/Card.test.tsx
import { render } from '@testing-library/react';
import { Card } from './Card';

describe('Card', () => {
  it('matches snapshot', () => {
    const { container } = render(
      <Card title="Test Card" description="Test description">
        Content
      </Card>
    );

    expect(container).toMatchSnapshot();
  });
});
```

### 11. Test Utilities

**Custom Render:**
```typescript
// tests/utils/test-utils.tsx
import { render, RenderOptions } from '@testing-library/react';
import { ReactElement } from 'react';
import { BrowserRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const AllTheProviders = ({ children }: { children: React.ReactNode }) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });

  return (
    <BrowserRouter>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </BrowserRouter>
  );
};

const customRender = (
  ui: ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => render(ui, { wrapper: AllTheProviders, ...options });

export * from '@testing-library/react';
export { customRender as render };
```

### 12. Coverage Reports

**package.json scripts:**
```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:watch": "vitest --watch"
  }
}
```

**CI Integration:**
```yaml
# .github/workflows/test.yml
name: Unit Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run test:coverage
      - uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

## Deliverables
- Comprehensive unit test suite
- Component tests with React Testing Library
- Hook tests
- Service/business logic tests
- API mocking with MSW
- >80% code coverage
- CI/CD integration
- Test utilities and helpers
- Documentation

## Testing Checklist
- [ ] All components have tests
- [ ] All custom hooks are tested
- [ ] Business logic has unit tests
- [ ] Edge cases are covered
- [ ] Error states are tested
- [ ] Loading states are tested
- [ ] Form validation is tested
- [ ] API calls are mocked
- [ ] Coverage thresholds are met
- [ ] Tests run in CI/CD
- [ ] Tests are fast (<10s total)
- [ ] No flaky tests
- [ ] Snapshots are minimal and meaningful
- [ ] Test names are descriptive
- [ ] Mocks are properly cleaned up
