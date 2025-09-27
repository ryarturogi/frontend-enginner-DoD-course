# Module 4: Frontend Testing in Practice

## 🎯 Learning Objectives

By the end of this module, you will:
- Master Jest and Vitest for unit testing React applications
- Use React Testing Library effectively for component integration tests
- Implement comprehensive E2E testing with Playwright and Cypress
- Set up accessibility testing with axe-core
- Configure visual regression testing with Chromatic and Percy
- Build a complete testing pipeline with proper tooling and configuration

## 🛠 Modern Testing Stack Overview

### Unit Testing: Jest vs Vitest

| Feature | Jest | Vitest |
|---------|------|--------|
| **Speed** | Good | Excellent (ESM native) |
| **Configuration** | Complex | Minimal |
| **Watch Mode** | Good | Excellent |
| **TypeScript** | Needs setup | Built-in |
| **ESM Support** | Limited | Native |
| **Bundle Size** | Larger | Smaller |
| **Ecosystem** | Mature | Growing |

### Integration Testing: React Testing Library

- **Philosophy:** Test behavior, not implementation
- **Queries:** Find elements like users do
- **Events:** Simulate real user interactions
- **Async:** Built-in async utilities

### E2E Testing: Playwright vs Cypress

| Feature | Playwright | Cypress |
|---------|------------|---------|
| **Browser Support** | Chrome, Firefox, Safari, Edge | Chrome, Firefox, Edge |
| **Language** | TypeScript/JavaScript | JavaScript |
| **Parallel Testing** | Built-in | Paid feature |
| **Network Stubbing** | Advanced | Built-in |
| **Mobile Testing** | Excellent | Limited |
| **Learning Curve** | Medium | Low |

## 🧪 Jest Setup and Configuration

### Installation and Basic Setup

```bash
pnpm add -D jest @types/jest jest-environment-jsdom
pnpm add -D @testing-library/jest-dom
```

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2)$': 'jest-transform-stub'
  },
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': ['babel-jest', {
      presets: [
        ['@babel/preset-env', { targets: { node: 'current' } }],
        ['@babel/preset-react', { runtime: 'automatic' }],
        '@babel/preset-typescript'
      ]
    }]
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/reportWebVitals.ts'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{test,spec}.{js,jsx,ts,tsx}'
  ]
};
```

### Setup File

```javascript
// src/setupTests.js
import '@testing-library/jest-dom';

// Mock console methods in tests
global.console = {
  ...console,
  // Uncomment to ignore specific console methods in tests
  // log: jest.fn(),
  // warn: jest.fn(),
  error: jest.fn(),
};

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};

// Mock ResizeObserver
global.ResizeObserver = class ResizeObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  unobserve() {}
};
```

### Advanced Jest Utilities

```javascript
// src/test-utils/jest-helpers.js
export const createMockComponent = (name) => {
  const MockComponent = (props) => (
    <div data-testid={`mock-${name.toLowerCase()}`} {...props} />
  );
  MockComponent.displayName = `Mock${name}`;
  return MockComponent;
};

export const createAsyncMock = (resolveValue, delay = 0) => {
  return jest.fn().mockImplementation(() => 
    new Promise(resolve => 
      setTimeout(() => resolve(resolveValue), delay)
    )
  );
};

export const mockLocalStorage = () => {
  const localStorageMock = {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn(),
  };
  Object.defineProperty(window, 'localStorage', {
    value: localStorageMock
  });
  return localStorageMock;
};

export const mockFetch = (responses) => {
  const mockFetch = jest.fn();
  
  responses.forEach((response, index) => {
    mockFetch.mockImplementationOnce(() =>
      Promise.resolve({
        ok: response.ok !== false,
        status: response.status || 200,
        json: () => Promise.resolve(response.data),
        text: () => Promise.resolve(JSON.stringify(response.data)),
      })
    );
  });
  
  global.fetch = mockFetch;
  return mockFetch;
};
```

## ⚡ Vitest Setup and Configuration

### Installation

```bash
pnpm add -D vitest @vitest/ui jsdom
pnpm add -D @testing-library/jest-dom
```

### Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./src/setupTests.ts'],
    globals: true,
    css: true,
    coverage: {
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/setupTests.ts',
        'src/test-utils/',
        '**/*.d.ts',
        '**/*.test.{ts,tsx}',
        '**/*.spec.{ts,tsx}'
      ],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80
        }
      }
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
});
```

### Vitest Utilities

```typescript
// src/test-utils/vitest-helpers.ts
import { vi } from 'vitest';

export const createMockTimer = () => {
  vi.useFakeTimers();
  return {
    advanceTime: (ms: number) => vi.advanceTimersByTime(ms),
    runAllTimers: () => vi.runAllTimers(),
    restore: () => vi.useRealTimers()
  };
};

export const createMockDate = (dateString: string) => {
  const mockDate = new Date(dateString);
  vi.setSystemTime(mockDate);
  return {
    restore: () => vi.useRealTimers()
  };
};

export const spyOnConsole = () => {
  const originalError = console.error;
  const errorSpy = vi.spyOn(console, 'error').mockImplementation(() => {});
  
  return {
    error: errorSpy,
    restore: () => {
      console.error = originalError;
    }
  };
};
```

## 📚 React Testing Library Deep Dive

### Core Principles and Best Practices

```typescript
// src/components/LoginForm/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  const defaultProps = {
    onSubmit: jest.fn(),
    onForgotPassword: jest.fn(),
    isLoading: false
  };

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('should render all form elements', () => {
    render(<LoginForm {...defaultProps} />);
    
    // Query by label text (accessible)
    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    
    // Query by role (semantic)
    expect(screen.getByRole('button', { name: /sign in/i })).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /forgot password/i })).toBeInTheDocument();
  });

  it('should validate email format', async () => {
    const user = userEvent.setup();
    render(<LoginForm {...defaultProps} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    const submitButton = screen.getByRole('button', { name: /sign in/i });
    
    // Enter invalid email
    await user.type(emailInput, 'invalid-email');
    await user.click(submitButton);
    
    // Check for validation message
    expect(await screen.findByText(/please enter a valid email/i)).toBeInTheDocument();
    expect(defaultProps.onSubmit).not.toHaveBeenCalled();
  });

  it('should submit form with valid data', async () => {
    const user = userEvent.setup();
    render(<LoginForm {...defaultProps} />);
    
    // Fill out form
    await user.type(screen.getByLabelText(/email/i), 'user@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /sign in/i }));
    
    // Verify submission
    await waitFor(() => {
      expect(defaultProps.onSubmit).toHaveBeenCalledWith({
        email: 'user@example.com',
        password: 'password123'
      });
    });
  });

  it('should show loading state', () => {
    render(<LoginForm {...defaultProps} isLoading={true} />);
    
    const submitButton = screen.getByRole('button', { name: /signing in/i });
    expect(submitButton).toBeDisabled();
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
  });

  it('should handle forgot password click', async () => {
    const user = userEvent.setup();
    render(<LoginForm {...defaultProps} />);
    
    await user.click(screen.getByRole('button', { name: /forgot password/i }));
    
    expect(defaultProps.onForgotPassword).toHaveBeenCalled();
  });
});
```

### Custom Render Function

```typescript
// src/test-utils/render.tsx
import { ReactElement } from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ThemeProvider } from '@/contexts/ThemeContext';
import { AuthProvider } from '@/contexts/AuthContext';

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  initialRoute?: string;
  queryClient?: QueryClient;
  authState?: {
    user: User | null;
    isAuthenticated: boolean;
  };
}

const createWrapper = (options: CustomRenderOptions = {}) => {
  const { 
    initialRoute = '/', 
    queryClient = new QueryClient({
      defaultOptions: {
        queries: { retry: false },
        mutations: { retry: false }
      }
    }),
    authState
  } = options;

  return ({ children }: { children: React.ReactNode }) => (
    <BrowserRouter>
      <QueryClientProvider client={queryClient}>
        <ThemeProvider>
          <AuthProvider initialState={authState}>
            {children}
          </AuthProvider>
        </ThemeProvider>
      </QueryClientProvider>
    </BrowserRouter>
  );
};

export const customRender = (
  ui: ReactElement,
  options: CustomRenderOptions = {}
) => {
  return render(ui, {
    wrapper: createWrapper(options),
    ...options
  });
};

// Re-export everything
export * from '@testing-library/react';
export { customRender as render };
```

### Testing Hooks

```typescript
// src/hooks/useShoppingCart.test.ts
import { renderHook, act } from '@testing-library/react';
import { useShoppingCart } from './useShoppingCart';

describe('useShoppingCart', () => {
  it('should initialize with empty cart', () => {
    const { result } = renderHook(() => useShoppingCart());
    
    expect(result.current.items).toEqual([]);
    expect(result.current.totalItems).toBe(0);
    expect(result.current.totalPrice).toBe(0);
  });

  it('should add items to cart', () => {
    const { result } = renderHook(() => useShoppingCart());
    const product = { id: 1, name: 'Test Product', price: 29.99 };
    
    act(() => {
      result.current.addItem(product);
    });
    
    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0]).toEqual({
      ...product,
      quantity: 1
    });
    expect(result.current.totalPrice).toBe(29.99);
  });

  it('should increase quantity for existing items', () => {
    const { result } = renderHook(() => useShoppingCart());
    const product = { id: 1, name: 'Test Product', price: 29.99 };
    
    act(() => {
      result.current.addItem(product);
      result.current.addItem(product);
    });
    
    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(2);
    expect(result.current.totalPrice).toBe(59.98);
  });

  it('should remove items from cart', () => {
    const { result } = renderHook(() => useShoppingCart());
    const product = { id: 1, name: 'Test Product', price: 29.99 };
    
    act(() => {
      result.current.addItem(product);
    });
    
    expect(result.current.items).toHaveLength(1);
    
    act(() => {
      result.current.removeItem(product.id);
    });
    
    expect(result.current.items).toHaveLength(0);
    expect(result.current.totalPrice).toBe(0);
  });
});
```

### Testing Context Providers

```typescript
// src/contexts/AuthContext.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { AuthProvider, useAuth } from './AuthContext';

// Test component to access context
const TestComponent = () => {
  const { user, login, logout, isLoading } = useAuth();
  
  return (
    <div>
      <div data-testid="user">{user?.name || 'Not logged in'}</div>
      <div data-testid="loading">{isLoading ? 'Loading' : 'Not loading'}</div>
      <button onClick={() => login('test@example.com', 'password')}>
        Login
      </button>
      <button onClick={logout}>Logout</button>
    </div>
  );
};

describe('AuthContext', () => {
  it('should provide initial state', () => {
    render(
      <AuthProvider>
        <TestComponent />
      </AuthProvider>
    );
    
    expect(screen.getByTestId('user')).toHaveTextContent('Not logged in');
    expect(screen.getByTestId('loading')).toHaveTextContent('Not loading');
  });

  it('should handle login flow', async () => {
    // Mock API call
    const mockLoginAPI = jest.fn().mockResolvedValue({
      user: { id: 1, name: 'John Doe', email: 'test@example.com' },
      token: 'jwt-token'
    });
    
    render(
      <AuthProvider loginAPI={mockLoginAPI}>
        <TestComponent />
      </AuthProvider>
    );
    
    fireEvent.click(screen.getByRole('button', { name: /login/i }));
    
    // Should show loading state
    expect(screen.getByTestId('loading')).toHaveTextContent('Loading');
    
    // Wait for login to complete
    await waitFor(() => {
      expect(screen.getByTestId('user')).toHaveTextContent('John Doe');
      expect(screen.getByTestId('loading')).toHaveTextContent('Not loading');
    });
  });

  it('should handle logout', async () => {
    render(
      <AuthProvider 
        initialState={{ 
          user: { id: 1, name: 'John Doe' }, 
          isAuthenticated: true 
        }}
      >
        <TestComponent />
      </AuthProvider>
    );
    
    expect(screen.getByTestId('user')).toHaveTextContent('John Doe');
    
    fireEvent.click(screen.getByRole('button', { name: /logout/i }));
    
    await waitFor(() => {
      expect(screen.getByTestId('user')).toHaveTextContent('Not logged in');
    });
  });
});
```

## 🎭 Playwright E2E Testing

### Installation and Setup

```bash
pnpm add -D @playwright/test
npx playwright install
```

### Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/results.xml' }]
  ],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure'
  },
  projects: [
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
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  webServer: {
    command: 'pnpm start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Page Object Model

```typescript
// e2e/pages/LoginPage.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly forgotPasswordLink: Locator;
  readonly errorMessage: Locator;
  readonly loadingSpinner: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel('Email');
    this.passwordInput = page.getByLabel('Password');
    this.loginButton = page.getByRole('button', { name: 'Sign In' });
    this.forgotPasswordLink = page.getByRole('link', { name: 'Forgot Password?' });
    this.errorMessage = page.getByTestId('error-message');
    this.loadingSpinner = page.getByTestId('loading-spinner');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async waitForLoadingToFinish() {
    await this.loadingSpinner.waitFor({ state: 'hidden' });
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}
```

### Comprehensive E2E Tests

```typescript
// e2e/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('User Authentication', () => {
  let loginPage: LoginPage;
  let dashboardPage: DashboardPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    dashboardPage = new DashboardPage(page);
    await loginPage.goto();
  });

  test('should login with valid credentials', async ({ page }) => {
    await loginPage.login('user@example.com', 'validpassword');
    
    // Wait for redirect
    await page.waitForURL('/dashboard');
    
    // Verify successful login
    await expect(dashboardPage.welcomeMessage).toBeVisible();
    await expect(dashboardPage.welcomeMessage).toContainText('Welcome back');
  });

  test('should show error for invalid credentials', async () => {
    await loginPage.login('user@example.com', 'wrongpassword');
    
    await expect(loginPage.errorMessage).toBeVisible();
    await expect(loginPage.errorMessage).toContainText('Invalid credentials');
  });

  test('should validate required fields', async () => {
    await loginPage.loginButton.click();
    
    // Check for validation errors
    await expect(loginPage.emailInput).toHaveAttribute('aria-invalid', 'true');
    await expect(loginPage.passwordInput).toHaveAttribute('aria-invalid', 'true');
  });

  test('should handle network errors gracefully', async ({ page }) => {
    // Simulate network failure
    await page.route('**/api/auth/login', route => {
      route.abort('failed');
    });
    
    await loginPage.login('user@example.com', 'password');
    
    await expect(loginPage.errorMessage).toContainText('Network error');
  });
});
```

### API Testing with Playwright

```typescript
// e2e/api/users.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Users API', () => {
  test('should create a new user', async ({ request }) => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      password: 'securepassword123'
    };
    
    const response = await request.post('/api/users', {
      data: userData
    });
    
    expect(response.ok()).toBeTruthy();
    
    const user = await response.json();
    expect(user.id).toBeDefined();
    expect(user.email).toBe(userData.email);
    expect(user.password).toBeUndefined(); // Should not return password
  });

  test('should get user profile', async ({ request }) => {
    // First create a user and get auth token
    const authResponse = await request.post('/api/auth/login', {
      data: {
        email: 'existing@example.com',
        password: 'password123'
      }
    });
    
    const { token } = await authResponse.json();
    
    // Get user profile
    const profileResponse = await request.get('/api/users/profile', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });
    
    expect(profileResponse.ok()).toBeTruthy();
    
    const profile = await profileResponse.json();
    expect(profile.email).toBe('existing@example.com');
  });

  test('should handle unauthorized access', async ({ request }) => {
    const response = await request.get('/api/users/profile');
    
    expect(response.status()).toBe(401);
    
    const error = await response.json();
    expect(error.message).toContain('Unauthorized');
  });
});
```

## 🌊 Cypress Alternative Setup

### Installation

```bash
pnpm add -D cypress @cypress/react @cypress/webpack-preprocessor
```

### Cypress Configuration

```typescript
// cypress.config.ts
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/e2e.ts'
  },
  component: {
    devServer: {
      framework: 'react',
      bundler: 'webpack',
    },
    specPattern: 'src/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/component.ts'
  },
});
```

### Cypress Commands

```typescript
// cypress/support/commands.ts
declare global {
  namespace Cypress {
    interface Chainable {
      login(email: string, password: string): Chainable<void>;
      dataCy(selector: string): Chainable<JQuery<HTMLElement>>;
      seedDatabase(): Chainable<void>;
      clearDatabase(): Chainable<void>;
    }
  }
}

Cypress.Commands.add('login', (email: string, password: string) => {
  cy.session([email, password], () => {
    cy.visit('/login');
    cy.get('[data-cy=email-input]').type(email);
    cy.get('[data-cy=password-input]').type(password);
    cy.get('[data-cy=login-button]').click();
    cy.url().should('include', '/dashboard');
  });
});

Cypress.Commands.add('dataCy', (selector: string) => {
  return cy.get(`[data-cy=${selector}]`);
});

Cypress.Commands.add('seedDatabase', () => {
  cy.request('POST', '/api/test/seed-database');
});

Cypress.Commands.add('clearDatabase', () => {
  cy.request('POST', '/api/test/clear-database');
});
```

### Cypress Tests

```typescript
// cypress/e2e/shopping-cart.cy.ts
describe('Shopping Cart', () => {
  beforeEach(() => {
    cy.clearDatabase();
    cy.seedDatabase();
    cy.login('customer@example.com', 'password123');
  });

  it('should add products to cart', () => {
    cy.visit('/products');
    
    // Add first product
    cy.dataCy('product-card').first().within(() => {
      cy.dataCy('add-to-cart-button').click();
    });
    
    // Verify cart count
    cy.dataCy('cart-count').should('contain', '1');
    
    // Add second product
    cy.dataCy('product-card').eq(1).within(() => {
      cy.dataCy('add-to-cart-button').click();
    });
    
    cy.dataCy('cart-count').should('contain', '2');
  });

  it('should complete checkout process', () => {
    // Add products to cart
    cy.visit('/products');
    cy.dataCy('product-card').first().within(() => {
      cy.dataCy('add-to-cart-button').click();
    });
    
    // Go to cart
    cy.dataCy('cart-icon').click();
    cy.url().should('include', '/cart');
    
    // Proceed to checkout
    cy.dataCy('checkout-button').click();
    cy.url().should('include', '/checkout');
    
    // Fill shipping information
    cy.dataCy('shipping-address').type('123 Test Street');
    cy.dataCy('shipping-city').type('Test City');
    cy.dataCy('shipping-state').select('CA');
    cy.dataCy('shipping-zip').type('12345');
    
    // Fill payment information
    cy.dataCy('card-number').type('4111111111111111');
    cy.dataCy('card-expiry').type('12/25');
    cy.dataCy('card-cvv').type('123');
    
    // Place order
    cy.dataCy('place-order-button').click();
    
    // Verify success
    cy.url().should('include', '/order-confirmation');
    cy.dataCy('order-status').should('contain', 'confirmed');
  });

  it('should handle cart persistence', () => {
    cy.visit('/products');
    
    // Add product to cart
    cy.dataCy('product-card').first().within(() => {
      cy.dataCy('add-to-cart-button').click();
    });
    
    // Refresh page
    cy.reload();
    
    // Cart should persist
    cy.dataCy('cart-count').should('contain', '1');
  });
});
```

## ♿ Accessibility Testing with axe-core

### Installation

```bash
pnpm add -D @axe-core/react axe-playwright cypress-axe
```

### Jest + React Testing Library Integration

```typescript
// src/test-utils/axe-helpers.ts
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

export const renderAndTestA11y = async (component: React.ReactElement) => {
  const { container } = render(component);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
  return { container };
};

export const testA11y = async (container: HTMLElement) => {
  const results = await axe(container);
  expect(results).toHaveNoViolations();
};
```

### Accessibility Tests

```typescript
// src/components/Button/Button.a11y.test.tsx
import { renderAndTestA11y, testA11y } from '@/test-utils/axe-helpers';
import { Button } from './Button';

describe('Button Accessibility', () => {
  it('should not have accessibility violations', async () => {
    await renderAndTestA11y(<Button>Click me</Button>);
  });

  it('should handle disabled state accessibly', async () => {
    await renderAndTestA11y(<Button disabled>Disabled Button</Button>);
  });

  it('should handle loading state accessibly', async () => {
    await renderAndTestA11y(
      <Button isLoading>
        Loading Button
      </Button>
    );
  });

  it('should support ARIA attributes', async () => {
    await renderAndTestA11y(
      <Button 
        aria-label="Close dialog"
        aria-describedby="close-help"
      >
        ×
      </Button>
    );
  });
});
```

### Playwright Accessibility Testing

```typescript
// e2e/accessibility/navigation.spec.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Navigation Accessibility', () => {
  test('should not have accessibility violations on home page', async ({ page }) => {
    await page.goto('/');
    
    const accessibilityScanResults = await new AxeBuilder({ page }).analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('should not have violations in navigation menu', async ({ page }) => {
    await page.goto('/');
    
    // Open navigation menu
    await page.click('[aria-label="Open navigation menu"]');
    
    const accessibilityScanResults = await new AxeBuilder({ page })
      .include('[role="navigation"]')
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('should handle keyboard navigation', async ({ page }) => {
    await page.goto('/');
    
    // Test tab navigation
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toBeVisible();
    
    // Test skip links
    await page.keyboard.press('Tab');
    const focusedElement = await page.locator(':focus');
    await expect(focusedElement).toHaveText('Skip to main content');
    
    // Test main navigation
    await page.keyboard.press('Enter');
    await expect(page.locator('main')).toBeFocused();
  });
});
```

## 🎨 Visual Regression Testing

### Playwright Screenshots

```typescript
// e2e/visual/components.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Component Visual Tests', () => {
  test('should match button variants', async ({ page }) => {
    await page.goto('/storybook/button');
    
    // Primary button
    const primaryButton = page.locator('[data-testid="primary-button"]');
    await expect(primaryButton).toHaveScreenshot('primary-button.png');
    
    // Secondary button
    const secondaryButton = page.locator('[data-testid="secondary-button"]');
    await expect(secondaryButton).toHaveScreenshot('secondary-button.png');
    
    // Disabled button
    const disabledButton = page.locator('[data-testid="disabled-button"]');
    await expect(disabledButton).toHaveScreenshot('disabled-button.png');
  });

  test('should match form layouts', async ({ page }) => {
    await page.goto('/forms/contact');
    
    // Full form screenshot
    await expect(page).toHaveScreenshot('contact-form.png', {
      fullPage: true
    });
    
    // Form with errors
    await page.click('[data-testid="submit-button"]');
    await expect(page.locator('form')).toHaveScreenshot('contact-form-errors.png');
  });

  test('should match responsive layouts', async ({ page }) => {
    const breakpoints = [
      { width: 375, height: 667, name: 'mobile' },
      { width: 768, height: 1024, name: 'tablet' },
      { width: 1920, height: 1080, name: 'desktop' }
    ];

    for (const breakpoint of breakpoints) {
      await page.setViewportSize(breakpoint);
      await page.goto('/');
      
      await expect(page).toHaveScreenshot(`homepage-${breakpoint.name}.png`, {
        fullPage: true
      });
    }
  });
});
```

### Percy Integration

```typescript
// src/components/ProductCard/ProductCard.percy.test.tsx
import { render } from '@testing-library/react';
import percySnapshot from '@percy/playwright';

describe('ProductCard Percy Tests', () => {
  const sampleProduct = {
    id: 1,
    name: 'Wireless Headphones',
    price: 199.99,
    image: '/images/headphones.jpg',
    rating: 4.5
  };

  it('should capture ProductCard variations', async () => {
    // Standard product card
    const { container: standardContainer } = render(
      <ProductCard product={sampleProduct} />
    );
    await percySnapshot('ProductCard - Standard');

    // On sale product card
    const { container: saleContainer } = render(
      <ProductCard 
        product={{
          ...sampleProduct,
          originalPrice: 249.99,
          salePrice: 199.99
        }} 
      />
    );
    await percySnapshot('ProductCard - On Sale');

    // Out of stock product card
    const { container: outOfStockContainer } = render(
      <ProductCard 
        product={{
          ...sampleProduct,
          inStock: false
        }} 
      />
    );
    await percySnapshot('ProductCard - Out of Stock');
  });
});
```

## 🔧 Testing Pipeline Configuration

### Package.json Scripts

```json
{
  "scripts": {
    "test": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:watch": "vitest --watch",
    "test:ui": "vitest --ui",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:component": "cypress run --component",
    "test:cypress": "cypress run",
    "test:a11y": "jest --testNamePattern='Accessibility'",
    "test:visual": "playwright test --grep='Visual'",
    "test:all": "npm run test:coverage && npm run test:e2e && npm run test:a11y"
  }
}
```

### GitHub Actions CI

```yaml
# .github/workflows/test.yml
name: Test Suite

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run unit tests
        run: pnpm test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Build application
        run: pnpm build
      
      - name: Run E2E tests
        run: pnpm test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  accessibility-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Run accessibility tests
        run: pnpm test:a11y

  visual-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Run visual tests
        run: pnpm test:visual
        env:
          PERCY_TOKEN: ${{ secrets.PERCY_TOKEN }}
```

## 📝 Exercise: Complete Testing Setup

Build a comprehensive testing setup for a React component:

### Step 1: Component Selection
Choose a complex component (e.g., a data table, form wizard, or shopping cart).

### Step 2: Multi-Level Testing
- **Unit tests:** Test individual functions and component behavior
- **Integration tests:** Test component interactions
- **E2E tests:** Test complete user workflows
- **Accessibility tests:** Ensure WCAG compliance
- **Visual tests:** Capture UI regressions

### Step 3: Testing Tools Setup
Configure and integrate:
- Vitest or Jest for unit testing
- React Testing Library for component testing
- Playwright for E2E testing
- axe-core for accessibility testing
- Percy or Playwright screenshots for visual testing

### Step 4: CI/CD Integration
Set up automated testing in your deployment pipeline.

**Deliverable:** A fully tested component with comprehensive test coverage across all testing levels.

---

**Next:** [Module 5: Production-Grade Code Practices](../05-code-practices/) - Learn SOLID principles, error handling, and architecture patterns for frontend applications.