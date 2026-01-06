# E2E Testing - Playwright

## Objective
Implement comprehensive end-to-end testing using Playwright for web applications, covering critical user flows, cross-browser testing, and CI/CD integration.

## Technology Stack
- Playwright (latest version)
- TypeScript
- Node.js
- GitHub Actions or similar CI/CD
- Playwright Test Runner
- Playwright Inspector for debugging

## Requirements

### 1. Test Coverage

**Authentication Flows:**
- User registration
- Login/logout
- Password reset
- Email verification
- Session persistence

**Core User Flows:**
- Complete checkout process
- Search and filter functionality
- Form submissions
- File uploads
- CRUD operations on resources

**UI/UX Testing:**
- Responsive design across viewports
- Navigation and routing
- Modal interactions
- Dropdown menus
- Infinite scroll
- Drag and drop

**Integration Testing:**
- Third-party service integrations
- Payment flows
- Email sending verification
- API interactions

### 2. Project Structure
```
tests/
├── e2e/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   ├── signup.spec.ts
│   │   └── password-reset.spec.ts
│   ├── checkout/
│   │   ├── cart.spec.ts
│   │   ├── payment.spec.ts
│   │   └── order-confirmation.spec.ts
│   ├── user/
│   │   ├── profile.spec.ts
│   │   └── settings.spec.ts
│   └── admin/
│       └── dashboard.spec.ts
├── fixtures/
│   ├── auth.fixture.ts
│   ├── user.fixture.ts
│   └── product.fixture.ts
├── page-objects/
│   ├── LoginPage.ts
│   ├── DashboardPage.ts
│   ├── CheckoutPage.ts
│   └── BasePage.ts
├── helpers/
│   ├── api.helper.ts
│   ├── db.helper.ts
│   └── test-data.helper.ts
├── utils/
│   ├── wait.ts
│   └── assertions.ts
├── config/
│   └── playwright.config.ts
└── global-setup.ts
```

### 3. Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/junit.xml' }],
    ['list'],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    actionTimeout: 10000,
    navigationTimeout: 30000,
  },

  projects: [
    // Desktop browsers
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },

    // Mobile browsers
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },

    // Tablet
    {
      name: 'iPad',
      use: { ...devices['iPad Pro'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

### 4. Page Object Model

**Base Page:**
```typescript
// page-objects/BasePage.ts
import { Page } from '@playwright/test';

export class BasePage {
  constructor(protected page: Page) {}

  async goto(path: string) {
    await this.page.goto(path);
  }

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
  }

  async screenshot(name: string) {
    await this.page.screenshot({ path: `screenshots/${name}.png` });
  }
}
```

**Login Page:**
```typescript
// page-objects/LoginPage.ts
import { Page, Locator } from '@playwright/test';
import { BasePage } from './BasePage';

export class LoginPage extends BasePage {
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    super(page);
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.submitButton = page.locator('[data-testid="submit-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  async goto() {
    await super.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}
```

### 5. Test Examples

**Login Test:**
```typescript
// tests/e2e/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../../page-objects/LoginPage';
import { DashboardPage } from '../../page-objects/DashboardPage';

test.describe('Login Flow', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123');

    const dashboardPage = new DashboardPage(page);
    await expect(page).toHaveURL('/dashboard');
    await expect(dashboardPage.welcomeMessage).toBeVisible();
  });

  test('should show error with invalid credentials', async () => {
    await loginPage.login('wrong@example.com', 'wrongpass');

    await expect(loginPage.errorMessage).toBeVisible();
    const errorText = await loginPage.getErrorMessage();
    expect(errorText).toContain('Invalid credentials');
  });

  test('should validate email format', async () => {
    await loginPage.emailInput.fill('invalid-email');
    await loginPage.submitButton.click();

    await expect(loginPage.emailInput).toHaveAttribute('aria-invalid', 'true');
  });
});
```

**Checkout Flow:**
```typescript
// tests/e2e/checkout/checkout.spec.ts
import { test, expect } from '@playwright/test';
import { CartPage } from '../../page-objects/CartPage';
import { CheckoutPage } from '../../page-objects/CheckoutPage';

test.describe('Checkout Process', () => {
  test.use({ storageState: 'auth.json' }); // Authenticated user

  test('should complete full checkout flow', async ({ page }) => {
    // Add item to cart
    await page.goto('/products/1');
    await page.click('[data-testid="add-to-cart"]');

    // Go to cart
    const cartPage = new CartPage(page);
    await cartPage.goto();
    await expect(cartPage.cartItems).toHaveCount(1);

    // Proceed to checkout
    await cartPage.proceedToCheckout();

    // Fill shipping info
    const checkoutPage = new CheckoutPage(page);
    await checkoutPage.fillShippingInfo({
      firstName: 'John',
      lastName: 'Doe',
      address: '123 Main St',
      city: 'New York',
      zipCode: '10001',
    });

    // Fill payment info (using Stripe test card)
    await checkoutPage.fillPaymentInfo({
      cardNumber: '4242424242424242',
      expiry: '12/25',
      cvc: '123',
    });

    // Complete order
    await checkoutPage.submitOrder();

    // Verify confirmation
    await expect(page).toHaveURL(/\/order-confirmation/);
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
  });
});
```

### 6. Fixtures & Helpers

**Auth Fixture:**
```typescript
// fixtures/auth.fixture.ts
import { test as base } from '@playwright/test';
import { LoginPage } from '../page-objects/LoginPage';

type AuthFixtures = {
  authenticatedPage: Page;
};

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('test@example.com', 'password123');
    await page.waitForURL('/dashboard');
    await use(page);
  },
});

export { expect } from '@playwright/test';
```

**API Helper:**
```typescript
// helpers/api.helper.ts
import { request } from '@playwright/test';

export class ApiHelper {
  static async createUser(userData: any) {
    const context = await request.newContext();
    const response = await context.post('/api/users', { data: userData });
    return response.json();
  }

  static async deleteUser(userId: string) {
    const context = await request.newContext();
    await context.delete(`/api/users/${userId}`);
  }
}
```

### 7. Advanced Features

**Visual Regression Testing:**
```typescript
test('should match homepage screenshot', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('homepage.png', {
    maxDiffPixels: 100,
  });
});
```

**Network Interception:**
```typescript
test('should handle API errors gracefully', async ({ page }) => {
  await page.route('**/api/products', (route) =>
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Internal Server Error' }),
    })
  );

  await page.goto('/products');
  await expect(page.locator('[data-testid="error-banner"]')).toBeVisible();
});
```

**File Upload:**
```typescript
test('should upload file successfully', async ({ page }) => {
  await page.goto('/upload');

  const fileInput = page.locator('input[type="file"]');
  await fileInput.setInputFiles('./test-files/sample.pdf');

  await page.click('[data-testid="upload-button"]');
  await expect(page.locator('[data-testid="success-message"]')).toBeVisible();
});
```

**Geolocation Testing:**
```typescript
test('should show location-based content', async ({ page, context }) => {
  await context.setGeolocation({ latitude: 40.7128, longitude: -74.006 });
  await context.grantPermissions(['geolocation']);

  await page.goto('/');
  await expect(page.locator('[data-testid="location"]')).toHaveText('New York');
});
```

### 8. CI/CD Integration

**GitHub Actions:**
```yaml
name: E2E Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright Browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### 9. Debugging

**Debug Mode:**
```bash
# Run tests in headed mode
npx playwright test --headed

# Run with debugger
npx playwright test --debug

# Run specific test file
npx playwright test tests/e2e/auth/login.spec.ts

# Run in UI mode
npx playwright test --ui
```

**Trace Viewer:**
```bash
# View trace for failed test
npx playwright show-trace trace.zip
```

### 10. Best Practices

**Selectors:**
- Use `data-testid` attributes for reliability
- Avoid CSS selectors that may change
- Use accessible selectors (roles, labels)

**Wait Strategies:**
- Use auto-waiting (Playwright built-in)
- Avoid fixed timeouts
- Wait for specific conditions

**Test Isolation:**
- Each test should be independent
- Clean up test data after tests
- Use fixtures for setup/teardown

**Performance:**
- Run tests in parallel
- Use storage state for authentication
- Mock external APIs when possible

## Deliverables
- Complete E2E test suite
- Page Object Model implementation
- Custom fixtures and helpers
- CI/CD integration
- Test reports and screenshots
- Documentation for running tests
- Visual regression tests
- API mocking examples

## Testing Checklist
- [ ] All critical user flows are covered
- [ ] Tests run on multiple browsers
- [ ] Mobile viewports are tested
- [ ] Authentication flows work correctly
- [ ] Form validations are tested
- [ ] Error states are covered
- [ ] Network failures are handled
- [ ] File uploads work
- [ ] Screenshots are captured on failure
- [ ] Tests run in CI/CD pipeline
- [ ] Test reports are generated
- [ ] Page objects are maintainable
- [ ] Tests are isolated and independent
- [ ] Visual regression tests pass
- [ ] Performance is acceptable
