# Module 4: Frontend Testing in Practice

## 🎯 Learning Objectives

By the end of this module, you will:
- **Master Modern Testing Architectures**: Implement Testing Trophy/Diamond patterns for optimal test distribution
- **Advanced Tool Proficiency**: Compare and implement Jest vs Vitest vs Node Test Runner with best practices
- **Next.js 15+ Testing Mastery**: Test App Router, Server Components, and modern React 19 patterns
- **AI-Powered Testing**: Integrate AI-powered testing tools (TestRigor, Applitools, LambdaTest KaneAI)
- **Modern E2E Excellence**: Master Playwright vs Cypress with advanced patterns and cross-browser testing
- **Contract Testing Implementation**: Set up consumer-driven contracts with Pact Framework
- **Performance Testing Integration**: Implement Core Web Vitals testing (LCP, INP, CLS) within test suites
- **Enterprise-Grade Accessibility**: Automate WCAG 2.1 AA compliance with modern a11y tools
- **Advanced Mocking Strategies**: Master MSW 2.0 and modern mocking patterns for realistic API simulation
- **Visual AI Testing**: Implement intelligent visual regression testing with self-healing capabilities
- **Property-Based Testing**: Use fast-check for comprehensive edge case discovery
- **Micro-Frontend Testing**: Test distributed architectures and cross-application integration
- **Security Testing Integration**: Include XSS, CSP, and vulnerability testing in frontend pipelines
- **Testing Observability**: Implement comprehensive testing metrics and quality gates
- **CI/CD Excellence**: Build modern testing pipelines with parallel execution and intelligent test selection

## 🏆 Testing Architecture Evolution

### From Testing Pyramid to Testing Trophy/Diamond

**Modern Enterprise Distribution (Recommended):**
- **Static Analysis (40%)**: TypeScript strict mode, ESLint, Prettier, Security scanning
- **Integration Tests (50%)**: Component integration, API contracts, User workflows
- **Unit Tests (20%)**: Pure functions, utilities, business logic
- **E2E Tests (10%)**: Critical user journeys, Smoke tests

### Testing Tools Comprehensive Comparison

| Feature | Jest | Vitest | Node Test Runner | Playwright | Cypress |
|---------|------|--------|------------------|------------|----------|
| **Performance** | Good | Excellent | Very Good | Excellent | Good |
| **ESM Support** | Limited | Native | Native | Native | Good |
| **TypeScript** | Setup Required | Built-in | Built-in | Built-in | Setup Required |
| **Vite Integration** | Not Supported | Native | Manual | Not Applicable | Manual |
| **Multi-browser** | Not Applicable | Not Applicable | Not Applicable | Chrome/Firefox/Safari/Edge | Chrome/Firefox/Edge |
| **Mobile Testing** | Not Applicable | Not Applicable | Not Applicable | Excellent | Limited |
| **AI Integration** | Limited | Growing | Minimal | Advanced | Growing |
| **Parallel Execution** | Good | Excellent | Good | Built-in | Paid |
| **Enterprise Adoption** | High | Rapidly Growing | Emerging | Increasing | Established |
| **Learning Curve** | Medium | Low | Low | Medium | Low |
| **Recommendation** | Legacy Projects | New Projects | Minimal Setups | E2E Testing | Component Testing |

### Modern React Testing Library Evolution

**Core Principles:**
- **Accessibility-First Queries**: Prioritize `getByRole`, `getByLabelText` for inclusive testing
- **User-Centric Testing**: Test actual user interactions, not implementation details
- **React 19 Support**: Native concurrent features, Actions, Suspense, and Server Components
- **AI-Assisted Element Discovery**: Intelligent element finding with self-healing selectors
- **Performance Integration**: Built-in performance testing with React DevTools integration

### Next.js 15+ Testing Challenges & Solutions

**Key Considerations:**
- **Server Components Testing**: "Use End-to-End Testing over Unit Testing for async components"
- **App Router Complexity**: Multi-layered testing approach for modern routing
- **RSC Hydration**: Testing server-rendered content with client-side interactions
- **Performance Testing**: Integration of Core Web Vitals in component tests

### AI-Powered Testing Revolution

**Leading Platforms:**
- **Applitools**: Visual AI with self-healing capabilities
- **TestRigor**: Natural language test creation (Inc. 5000 recognition)
- **LambdaTest KaneAI**: LLM-powered E2E test generation
- **Testsigma**: Codeless, AI-powered test automation
- **Market Growth**: $3.4 billion projected by 2033 (19% CAGR)

### Modern Mocking Evolution: MSW 2.0+

**Revolutionary Features:**
- **Environment Agnostic**: Works seamlessly across browsers and Node.js
- **Service Worker Approach**: Network-level interception without code patching
- **Universal Compatibility**: Works with fetch, Axios, React Query, Apollo
- **Reusable API Layer**: Consistent mocking across development, testing, E2E, and Storybook
- **Performance Benefits**: Faster test execution with realistic network simulation

## 🏗️ Modern Testing Infrastructure Setup

### Testing Architecture Decision Matrix

```typescript
// Choose your testing stack based on project requirements
interface TestingStackDecision {
  projectType: 'new' | 'legacy' | 'enterprise' | 'minimal';
  framework: 'react' | 'nextjs' | 'micro-frontend';
  team: 'small' | 'medium' | 'large';
  timeline: 'rapid' | 'standard' | 'comprehensive';
}

const getRecommendedStack = (requirements: TestingStackDecision): TestingStack => {
  // Decision Logic
  if (requirements.projectType === 'new' && requirements.framework === 'nextjs') {
    return {
      unitTesting: 'vitest',
      e2eTesting: 'playwright',
      componentTesting: 'testing-library + vitest',
      visualTesting: 'playwright-screenshots',
      apiMocking: 'msw-2.0',
      aiIntegration: 'applitools',
    };
  }
  
  if (requirements.projectType === 'enterprise') {
    return {
      unitTesting: 'vitest',
      e2eTesting: 'playwright',
      contractTesting: 'pact',
      performanceTesting: 'lighthouse-ci',
      accessibilityTesting: 'axe-core',
      securityTesting: 'eslint-plugin-security',
      visualTesting: 'applitools',
      aiIntegration: 'testRigor',
    };
  }
  
  // Legacy project recommendations
  return {
    unitTesting: 'jest', // Maintain existing setup
    e2eTesting: 'cypress', // Easier migration
    visualTesting: 'percy',
    incrementalMigration: true,
  };
};
```

## ⚡ Vitest: The Testing Standard

### Enterprise-Grade Installation

```bash
# Modern Vitest setup with comprehensive tooling
pnpm add -D vitest @vitest/ui @vitest/coverage-v8
pnpm add -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
pnpm add -D jsdom happy-dom # Choose based on compatibility needs
pnpm add -D msw @mswjs/data # Modern API mocking
pnpm add -D @axe-core/react # Accessibility testing
pnpm add -D @playwright/test # E2E testing
pnpm add -D fast-check # Property-based testing
pnpm add -D @pact-foundation/pact # Contract testing
```

### Advanced Vitest Configuration

```typescript
// vitest.config.ts - Production-ready configuration
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';
import { configDefaults } from 'vitest/config';

export default defineConfig({
  plugins: [react()],
  test: {
    // Environment selection based on needs
    environment: 'happy-dom', // Faster than jsdom for most cases
    
    // Global setup and teardown
    setupFiles: ['./src/setupTests.ts'],
    globalSetup: ['./src/globalSetup.ts'],
    
    // Enhanced globals for better DX
    globals: true,
    css: true,
    
    // Performance optimization
    pool: 'threads',
    poolOptions: {
      threads: {
        singleThread: false,
        isolate: false, // Faster execution
      },
    },
    
    // Advanced coverage configuration
    coverage: {
      provider: 'v8', // Fastest coverage provider
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        ...configDefaults.coverage.exclude,
        'src/setupTests.ts',
        'src/test-utils/',
        'src/**/*.stories.{ts,tsx}',
        'src/**/*.d.ts',
        'e2e/**',
        'playwright.config.ts',
      ],
      thresholds: {
        global: {
          branches: 85, // Higher standard
          functions: 85,
          lines: 85,
          statements: 85,
        },
        // Per-file thresholds for critical modules
        './src/utils/': {
          branches: 95,
          functions: 95,
          lines: 95,
          statements: 95,
        },
      },
    },
    
    // Test filtering and organization
    include: ['src/**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
    exclude: [
      ...configDefaults.exclude,
      'e2e/**',
    ],
    
    // Timeout configuration
    testTimeout: 10000,
    hookTimeout: 10000,
    
    // Watch mode optimization
    watch: {
      ignore: ['dist/**', 'node_modules/**', '.git/**'],
    },
    
    // Advanced reporters
    reporters: [
      'default',
      'json',
      ['html', { outputFile: 'test-report.html' }],
      // CI-specific reporters
      ...(process.env.CI ? [['junit', { outputFile: 'test-results.xml' }]] : []),
    ],
    
    // Benchmark testing for performance-critical code
    benchmark: {
      include: ['src/**/*.{bench,benchmark}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
      exclude: ['node_modules', 'dist', '.git'],
    },
  },
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@test-utils': path.resolve(__dirname, './src/test-utils'),
      '@fixtures': path.resolve(__dirname, './src/__fixtures__'),
    },
  },
  
  // Environment variables for testing
  define: {
    __TEST_ENV__: true,
    'import.meta.env.VITE_API_URL': JSON.stringify('http://localhost:3001/api'),
  },
});
```

### Modern Setup Files

```typescript
// src/setupTests.ts - Comprehensive test environment setup
import '@testing-library/jest-dom';
import { vi } from 'vitest';
import { server } from './mocks/server';
import { cleanup } from '@testing-library/react';

// MSW server setup
beforeAll(() => {
  server.listen({
    onUnhandledRequest: 'error', // Fail tests on unhandled requests
  });
});

afterEach(() => {
  server.resetHandlers();
  cleanup(); // Automatic cleanup
});

afterAll(() => {
  server.close();
});

// Enhanced global mocks
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(), // deprecated
    removeListener: vi.fn(), // deprecated
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Modern Web APIs
Object.defineProperty(window, 'IntersectionObserver', {
  writable: true,
  value: vi.fn().mockImplementation(() => ({
    observe: vi.fn(),
    unobserve: vi.fn(),
    disconnect: vi.fn(),
    root: null,
    rootMargin: '',
    thresholds: [],
  })),
});

Object.defineProperty(window, 'ResizeObserver', {
  writable: true,
  value: vi.fn().mockImplementation(() => ({
    observe: vi.fn(),
    unobserve: vi.fn(),
    disconnect: vi.fn(),
  })),
});

// Web Performance APIs
Object.defineProperty(window, 'performance', {
  writable: true,
  value: {
    ...window.performance,
    mark: vi.fn(),
    measure: vi.fn(),
    getEntriesByName: vi.fn(() => []),
    getEntriesByType: vi.fn(() => []),
    navigation: {
      type: 'navigate',
      redirectCount: 0,
    },
  },
});

// Clipboard API
Object.defineProperty(navigator, 'clipboard', {
  writable: true,
  value: {
    writeText: vi.fn(),
    readText: vi.fn(),
  },
});

// Geolocation API
Object.defineProperty(navigator, 'geolocation', {
  writable: true,
  value: {
    getCurrentPosition: vi.fn(),
    watchPosition: vi.fn(),
    clearWatch: vi.fn(),
  },
});

// Web Workers
class MockWorker {
  constructor(public url: string) {}
  addEventListener = vi.fn();
  removeEventListener = vi.fn();
  postMessage = vi.fn();
  terminate = vi.fn();
  onmessage = null;
  onerror = null;
}

Object.defineProperty(window, 'Worker', {
  writable: true,
  value: MockWorker,
});

// Service Worker
Object.defineProperty(navigator, 'serviceWorker', {
  writable: true,
  value: {
    register: vi.fn(),
    getRegistration: vi.fn(),
    ready: Promise.resolve({
      unregister: vi.fn(),
      update: vi.fn(),
    }),
  },
});

// Global test utilities
declare global {
  var testUtils: {
    createMockDate: (dateString: string) => { restore: () => void };
    mockConsoleMethod: (method: 'log' | 'warn' | 'error') => vi.MockedFunction<any>;
    flushPromises: () => Promise<void>;
  };
}

global.testUtils = {
  createMockDate: (dateString: string) => {
    const mockDate = new Date(dateString);
    vi.setSystemTime(mockDate);
    return {
      restore: () => vi.useRealTimers(),
    };
  },
  
  mockConsoleMethod: (method: 'log' | 'warn' | 'error') => {
    return vi.spyOn(console, method).mockImplementation(() => {});
  },
  
  flushPromises: () => new Promise(resolve => setTimeout(resolve, 0)),
};
```

## 🧬 MSW 2.0: Next-Generation API Mocking

### Revolutionary MSW 2.0 Setup

```bash
# MSW 2.0 with enhanced features
pnpm add -D msw @mswjs/data @mswjs/http-middleware
```

### Advanced MSW 2.0 Implementation

```typescript
// src/mocks/handlers.ts - Enterprise-grade API mocking
import { http, HttpResponse, delay } from 'msw';
import { factory, primaryKey } from '@mswjs/data';

// Database factory for realistic data relationships
const db = factory({
  user: {
    id: primaryKey(() => `user-${Math.random().toString(36).substr(2, 9)}`),
    name: () => 'John Doe',
    email: () => 'john@example.com',
    avatar: () => 'https://via.placeholder.com/150',
    role: () => 'user',
    createdAt: () => new Date().toISOString(),
    preferences: () => ({
      theme: 'light',
      notifications: true,
      language: 'en',
    }),
  },
  product: {
    id: primaryKey(() => `product-${Math.random().toString(36).substr(2, 9)}`),
    name: () => 'Sample Product',
    price: () => Math.floor(Math.random() * 1000) + 10,
    category: () => 'electronics',
    inStock: () => Math.random() > 0.1,
    rating: () => Number((Math.random() * 4 + 1).toFixed(1)),
    reviews: () => Math.floor(Math.random() * 1000),
    createdAt: () => new Date().toISOString(),
  },
  order: {
    id: primaryKey(() => `order-${Math.random().toString(36).substr(2, 9)}`),
    userId: () => 'user-1',
    status: () => 'pending',
    items: () => [],
    total: () => 0,
    createdAt: () => new Date().toISOString(),
  },
});

// Seed initial data
db.user.create({ id: 'user-1', name: 'Test User', email: 'test@example.com' });
db.product.createMany([
  { name: 'Wireless Headphones', price: 199, category: 'audio' },
  { name: 'Smart Watch', price: 299, category: 'wearables' },
  { name: 'Laptop Stand', price: 49, category: 'accessories' },
]);

export const handlers = [
  // Authentication endpoints
  http.post('/api/auth/login', async ({ request }) => {
    const { email, password } = await request.json() as { email: string; password: string };
    
    // Simulate authentication delay
    await delay(800);
    
    if (email === 'test@example.com' && password === 'password123') {
      const user = db.user.findFirst({ where: { email: { equals: email } } });
      return HttpResponse.json({
        user,
        token: 'mock-jwt-token',
        expiresIn: 3600,
      });
    }
    
    return HttpResponse.json(
      { error: 'Invalid credentials', code: 'AUTH_FAILED' },
      { status: 401 }
    );
  }),

  http.post('/api/auth/logout', async () => {
    await delay(200);
    return HttpResponse.json({ message: 'Logged out successfully' });
  }),

  // User endpoints
  http.get('/api/users/profile', ({ request }) => {
    const authHeader = request.headers.get('authorization');
    
    if (!authHeader?.startsWith('Bearer ')) {
      return HttpResponse.json(
        { error: 'Unauthorized', code: 'NO_TOKEN' },
        { status: 401 }
      );
    }
    
    const user = db.user.findFirst({ where: { id: { equals: 'user-1' } } });
    return HttpResponse.json(user);
  }),

  http.patch('/api/users/profile', async ({ request }) => {
    const updates = await request.json() as Partial<User>;
    const user = db.user.update({
      where: { id: { equals: 'user-1' } },
      data: updates,
    });
    
    await delay(500);
    return HttpResponse.json(user);
  }),

  // Product endpoints with advanced filtering
  http.get('/api/products', ({ request }) => {
    const url = new URL(request.url);
    const page = parseInt(url.searchParams.get('page') || '1');
    const limit = parseInt(url.searchParams.get('limit') || '20');
    const category = url.searchParams.get('category');
    const search = url.searchParams.get('search');
    const sortBy = url.searchParams.get('sortBy') || 'createdAt';
    const sortOrder = url.searchParams.get('sortOrder') || 'desc';

    let products = db.product.findMany({});

    // Apply filters
    if (category) {
      products = products.filter(p => p.category === category);
    }

    if (search) {
      products = products.filter(p => 
        p.name.toLowerCase().includes(search.toLowerCase())
      );
    }

    // Apply sorting
    products.sort((a, b) => {
      const aVal = a[sortBy as keyof typeof a];
      const bVal = b[sortBy as keyof typeof b];
      
      if (sortOrder === 'asc') {
        return aVal > bVal ? 1 : -1;
      }
      return aVal < bVal ? 1 : -1;
    });

    // Apply pagination
    const total = products.length;
    const offset = (page - 1) * limit;
    const paginatedProducts = products.slice(offset, offset + limit);

    return HttpResponse.json({
      products: paginatedProducts,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    });
  }),

  http.get('/api/products/:id', ({ params }) => {
    const product = db.product.findFirst({
      where: { id: { equals: params.id as string } },
    });

    if (!product) {
      return HttpResponse.json(
        { error: 'Product not found', code: 'PRODUCT_NOT_FOUND' },
        { status: 404 }
      );
    }

    return HttpResponse.json(product);
  }),

  // Order endpoints
  http.post('/api/orders', async ({ request }) => {
    const orderData = await request.json() as CreateOrderRequest;
    
    // Simulate order processing delay
    await delay(1200);
    
    const order = db.order.create({
      ...orderData,
      status: 'confirmed',
    });

    return HttpResponse.json(order, { status: 201 });
  }),

  http.get('/api/orders/:id', ({ params }) => {
    const order = db.order.findFirst({
      where: { id: { equals: params.id as string } },
    });

    if (!order) {
      return HttpResponse.json(
        { error: 'Order not found', code: 'ORDER_NOT_FOUND' },
        { status: 404 }
      );
    }

    return HttpResponse.json(order);
  }),

  // Error simulation endpoints for testing
  http.get('/api/error/500', () => {
    return HttpResponse.json(
      { error: 'Internal server error', code: 'INTERNAL_ERROR' },
      { status: 500 }
    );
  }),

  http.get('/api/error/timeout', async () => {
    await delay(30000); // Simulate timeout
    return HttpResponse.json({ message: 'This will timeout' });
  }),

  http.get('/api/error/network', () => {
    return HttpResponse.error();
  }),

  // Performance testing endpoints
  http.get('/api/performance/slow', async () => {
    await delay(5000);
    return HttpResponse.json({ message: 'Slow response' });
  }),

  http.get('/api/performance/large-payload', () => {
    const largeData = Array(10000).fill(0).map((_, i) => ({
      id: i,
      data: `Large data item ${i}`,
      timestamp: new Date().toISOString(),
    }));

    return HttpResponse.json({ items: largeData });
  }),
];
```

### MSW Server Setup

```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

// Setup server for Node.js testing environment
export const server = setupServer(...handlers);

// src/mocks/browser.ts
import { setupWorker } from 'msw/browser';
import { handlers } from './handlers';

// Setup worker for browser development
export const worker = setupWorker(...handlers);
```

### Advanced MSW Testing Patterns

```typescript
// src/components/UserProfile/UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { server } from '@/mocks/server';
import { http, HttpResponse } from 'msw';
import { UserProfile } from './UserProfile';

describe('UserProfile with MSW', () => {
  const user = userEvent.setup();

  test('should display user profile data', async () => {
    render(<UserProfile userId="user-1" />);
    
    // MSW automatically handles the API call
    await waitFor(() => {
      expect(screen.getByText('Test User')).toBeInTheDocument();
      expect(screen.getByText('test@example.com')).toBeInTheDocument();
    });
  });

  test('should handle profile update', async () => {
    render(<UserProfile userId="user-1" />);
    
    // Wait for initial load
    await waitFor(() => {
      expect(screen.getByDisplayValue('Test User')).toBeInTheDocument();
    });
    
    // Update name
    const nameInput = screen.getByLabelText(/name/i);
    await user.clear(nameInput);
    await user.type(nameInput, 'Updated Name');
    
    // Submit update
    await user.click(screen.getByRole('button', { name: /save/i }));
    
    // Verify success message
    await waitFor(() => {
      expect(screen.getByText(/profile updated successfully/i)).toBeInTheDocument();
    });
  });

  test('should handle network errors gracefully', async () => {
    // Override handler for this specific test
    server.use(
      http.get('/api/users/profile', () => {
        return HttpResponse.json(
          { error: 'Server error', code: 'INTERNAL_ERROR' },
          { status: 500 }
        );
      })
    );

    render(<UserProfile userId="user-1" />);
    
    await waitFor(() => {
      expect(screen.getByText(/failed to load profile/i)).toBeInTheDocument();
      expect(screen.getByRole('button', { name: /retry/i })).toBeInTheDocument();
    });
  });

  test('should handle unauthorized access', async () => {
    server.use(
      http.get('/api/users/profile', () => {
        return HttpResponse.json(
          { error: 'Unauthorized', code: 'NO_TOKEN' },
          { status: 401 }
        );
      })
    );

    render(<UserProfile userId="user-1" />);
    
    await waitFor(() => {
      expect(screen.getByText(/please log in to continue/i)).toBeInTheDocument();
    });
  });

  test('should simulate slow network conditions', async () => {
    server.use(
      http.get('/api/users/profile', async () => {
        // Simulate slow network
        await delay(3000);
        return HttpResponse.json({ id: 'user-1', name: 'Test User' });
      })
    );

    render(<UserProfile userId="user-1" />);
    
    // Should show loading state
    expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
    
    // Wait for data to load
    await waitFor(
      () => {
        expect(screen.getByText('Test User')).toBeInTheDocument();
      },
      { timeout: 5000 }
    );
  });
});
```

## 🎭 Advanced React Testing Library Patterns

### Modern Component Testing with Accessibility Focus

```typescript
// src/components/SearchableProductList/SearchableProductList.test.tsx
import { render, screen, within, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { axe, toHaveNoViolations } from 'jest-axe';
import { SearchableProductList } from './SearchableProductList';
import { createWrapper } from '@test-utils/wrapper';

expect.extend(toHaveNoViolations);

describe('SearchableProductList', () => {
  const user = userEvent.setup();

  test('should render products with accessibility-first queries', async () => {
    const { container } = render(<SearchableProductList />, {
      wrapper: createWrapper(),
    });
    
    // Accessibility check
    const results = await axe(container);
    expect(results).toHaveNoViolations();
    
    // Use semantic queries
    expect(screen.getByRole('searchbox', { name: /search products/i })).toBeInTheDocument();
    expect(screen.getByRole('combobox', { name: /filter by category/i })).toBeInTheDocument();
    
    // Wait for products to load
    await waitFor(() => {
      const productList = screen.getByRole('list', { name: /product results/i });
      expect(productList).toBeInTheDocument();
      
      const products = within(productList).getAllByRole('listitem');
      expect(products.length).toBeGreaterThan(0);
    });
  });

  test('should handle search functionality with debouncing', async () => {
    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    const searchInput = screen.getByRole('searchbox', { name: /search products/i });
    
    // Type search query
    await user.type(searchInput, 'wireless headphones');
    
    // Should show loading state during debounce
    expect(screen.getByLabelText(/searching/i)).toBeInTheDocument();
    
    // Wait for search results
    await waitFor(() => {
      const results = screen.getAllByRole('listitem');
      expect(results).toHaveLength(1);
      
      const firstResult = results[0];
      expect(within(firstResult).getByText(/wireless headphones/i)).toBeInTheDocument();
    });
  });

  test('should handle empty search results', async () => {
    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    const searchInput = screen.getByRole('searchbox', { name: /search products/i });
    await user.type(searchInput, 'nonexistent product xyz');
    
    await waitFor(() => {
      expect(screen.getByText(/no products found/i)).toBeInTheDocument();
      expect(screen.getByText(/try adjusting your search/i)).toBeInTheDocument();
    });
  });

  test('should handle keyboard navigation', async () => {
    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    // Wait for products to load
    await waitFor(() => {
      expect(screen.getAllByRole('listitem')).toHaveLength(3);
    });
    
    const firstProduct = screen.getAllByRole('listitem')[0];
    const addToCartButton = within(firstProduct).getByRole('button', { name: /add to cart/i });
    
    // Test keyboard navigation
    addToCartButton.focus();
    expect(addToCartButton).toHaveFocus();
    
    // Press Enter to add to cart
    await user.keyboard('{Enter}');
    
    // Should show success message
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent(/added to cart/i);
    });
  });

  test('should handle error states gracefully', async () => {
    // Mock API failure
    server.use(
      http.get('/api/products', () => {
        return HttpResponse.error();
      })
    );

    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    await waitFor(() => {
      expect(screen.getByRole('alert')).toHaveTextContent(/failed to load products/i);
      expect(screen.getByRole('button', { name: /try again/i })).toBeInTheDocument();
    });
  });

  test('should maintain performance with large datasets', async () => {
    // Mock large dataset
    server.use(
      http.get('/api/products', () => {
        const largeProductList = Array(1000).fill(0).map((_, i) => ({
          id: `product-${i}`,
          name: `Product ${i}`,
          price: Math.floor(Math.random() * 1000) + 10,
          category: i % 2 === 0 ? 'electronics' : 'clothing',
        }));

        return HttpResponse.json({
          products: largeProductList,
          pagination: { page: 1, limit: 20, total: 1000, pages: 50 },
        });
      })
    );

    const startTime = performance.now();
    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    await waitFor(() => {
      expect(screen.getAllByRole('listitem')).toHaveLength(20); // Virtualized list
    });
    
    const renderTime = performance.now() - startTime;
    expect(renderTime).toBeLessThan(1000); // Should render within 1 second
  });

  test('should handle concurrent user interactions', async () => {
    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    const searchInput = screen.getByRole('searchbox');
    const categoryFilter = screen.getByRole('combobox', { name: /category/i });
    
    // Simulate concurrent user interactions
    await Promise.all([
      user.type(searchInput, 'laptop'),
      user.selectOptions(categoryFilter, 'electronics'),
    ]);
    
    await waitFor(() => {
      // Should apply both filters
      const results = screen.getAllByRole('listitem');
      expect(results.length).toBeGreaterThan(0);
      
      results.forEach(result => {
        expect(within(result).getByText(/laptop/i)).toBeInTheDocument();
      });
    });
  });

  test('should support screen reader announcements', async () => {
    render(<SearchableProductList />, { wrapper: createWrapper() });
    
    const searchInput = screen.getByRole('searchbox');
    await user.type(searchInput, 'headphones');
    
    await waitFor(() => {
      // Check for screen reader announcements
      const liveRegion = screen.getByRole('status');
      expect(liveRegion).toHaveTextContent(/found \d+ products/i);
    });
  });
});
```

### Advanced Hook Testing Patterns

```typescript
// src/hooks/useProductSearch.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useProductSearch } from './useProductSearch';
import { server } from '@/mocks/server';
import { http, HttpResponse } from 'msw';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

  return ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useProductSearch Hook', () => {
  test('should initialize with default state', () => {
    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    expect(result.current.products).toEqual([]);
    expect(result.current.isLoading).toBe(false);
    expect(result.current.error).toBeNull();
    expect(result.current.hasNextPage).toBe(false);
  });

  test('should search products with query', async () => {
    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    // Trigger search
    result.current.searchProducts('wireless headphones');

    expect(result.current.isLoading).toBe(true);

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
      expect(result.current.products).toHaveLength(1);
      expect(result.current.products[0]).toMatchObject({
        name: expect.stringContaining('Wireless Headphones'),
      });
    });
  });

  test('should handle search debouncing', async () => {
    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    // Rapid fire searches (should debounce)
    result.current.searchProducts('a');
    result.current.searchProducts('ab');
    result.current.searchProducts('abc');
    result.current.searchProducts('wireless');

    // Should only make one API call after debounce
    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    expect(result.current.query).toBe('wireless');
  });

  test('should support infinite pagination', async () => {
    server.use(
      http.get('/api/products', ({ request }) => {
        const url = new URL(request.url);
        const page = parseInt(url.searchParams.get('page') || '1');

        return HttpResponse.json({
          products: Array(20).fill(0).map((_, i) => ({
            id: `product-${page}-${i}`,
            name: `Product ${page}-${i}`,
            price: 100 + i,
          })),
          pagination: {
            page,
            limit: 20,
            total: 100,
            pages: 5,
          },
        });
      })
    );

    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    // Initial search
    result.current.searchProducts('all products');

    await waitFor(() => {
      expect(result.current.products).toHaveLength(20);
      expect(result.current.hasNextPage).toBe(true);
    });

    // Load next page
    result.current.fetchNextPage();

    await waitFor(() => {
      expect(result.current.products).toHaveLength(40);
    });
  });

  test('should handle concurrent searches', async () => {
    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    // Start two searches concurrently
    result.current.searchProducts('query1');
    result.current.searchProducts('query2');

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });

    // Should only show results for the last query
    expect(result.current.query).toBe('query2');
  });

  test('should handle network errors', async () => {
    server.use(
      http.get('/api/products', () => {
        return HttpResponse.error();
      })
    );

    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    result.current.searchProducts('failing query');

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
      expect(result.current.error).toBeTruthy();
      expect(result.current.products).toEqual([]);
    });
  });

  test('should support search filters', async () => {
    const { result } = renderHook(() => useProductSearch(), {
      wrapper: createWrapper(),
    });

    // Apply filters
    result.current.setFilters({
      category: 'electronics',
      minPrice: 100,
      maxPrice: 500,
    });

    result.current.searchProducts('laptop');

    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
      expect(result.current.appliedFilters).toEqual({
        category: 'electronics',
        minPrice: 100,
        maxPrice: 500,
      });
    });
  });
});
```

## 📚 Storybook: Component Development & Testing

### Storybook 8+ with Modern Features

Storybook has evolved into a comprehensive component development environment with advanced testing capabilities:

```typescript
// .storybook/main.ts - Modern Storybook 8+ configuration
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: ['../src/**/*.stories.@(js|jsx|mjs|ts|tsx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
    '@storybook/addon-viewport',
    '@storybook/addon-docs',
    '@storybook/addon-coverage',
    '@storybook/addon-test-runner',
    '@storybook/addon-storysource',
    '@storybook/addon-design-tokens',
  ],
  framework: {
    name: '@storybook/nextjs',
    options: {
      nextConfigPath: '../next.config.ts',
    },
  },
  typescript: {
    check: false,
    reactDocgen: 'react-docgen-typescript',
    reactDocgenTypescriptOptions: {
      shouldExtractLiteralValuesFromEnum: true,
      propFilter: (prop) => (prop.parent ? !/node_modules/.test(prop.parent.fileName) : true),
    },
  },
  features: {
    buildStoriesJson: true,
  },
  docs: {
    autodocs: 'tag',
  },
};

export default config;
```

### Component Stories with React 19 Features

```tsx
// src/components/UserForm.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { within, userEvent, expect } from '@storybook/test';
import { UserForm } from './UserForm';

const meta: Meta<typeof UserForm> = {
  title: 'Forms/UserForm',
  component: UserForm,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'A modern user form with React 19 Actions and optimistic updates.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    onSubmit: { action: 'submitted' },
    onError: { action: 'error' },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

// Default story
export const Default: Story = {
  args: {
    initialData: {
      name: '',
      email: '',
    },
  },
};

// Story with pre-filled data
export const PreFilled: Story = {
  args: {
    initialData: {
      name: 'John Doe',
      email: 'john@example.com',
    },
  },
};

// Interactive story with testing
export const InteractiveForm: Story = {
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);
    
    // Test form submission
    const nameInput = canvas.getByLabelText(/name/i);
    const emailInput = canvas.getByLabelText(/email/i);
    const submitButton = canvas.getByRole('button', { name: /submit/i });
    
    await userEvent.type(nameInput, 'Jane Smith');
    await userEvent.type(emailInput, 'jane@example.com');
    await userEvent.click(submitButton);
    
    // Verify loading state
    await expect(canvas.getByText(/submitting/i)).toBeInTheDocument();
    
    // Wait for success message
    await expect(canvas.getByText(/success/i)).toBeInTheDocument();
  },
};

// Error state story
export const WithValidationErrors: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const submitButton = canvas.getByRole('button', { name: /submit/i });
    
    await userEvent.click(submitButton);
    
    // Verify validation errors
    await expect(canvas.getByText(/name is required/i)).toBeInTheDocument();
    await expect(canvas.getByText(/email is required/i)).toBeInTheDocument();
  },
};

// Accessibility testing story
export const AccessibilityTest: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    
    // Test keyboard navigation
    await userEvent.tab();
    await expect(canvas.getByLabelText(/name/i)).toHaveFocus();
    
    await userEvent.tab();
    await expect(canvas.getByLabelText(/email/i)).toHaveFocus();
    
    await userEvent.tab();
    await expect(canvas.getByRole('button', { name: /submit/i })).toHaveFocus();
  },
};
```

### Visual Testing with Chromatic

```typescript
// .storybook/test-runner.ts
import type { TestRunnerConfig } from '@storybook/test-runner';
import { checkA11y, injectAxe } from 'axe-playwright';

const config: TestRunnerConfig = {
  setup() {
    // Inject axe for accessibility testing
    injectAxe();
  },
  async preVisit(page, context) {
    // Setup MSW for API mocking
    await page.addInitScript(() => {
      window.__MSW_START__ = true;
    });
  },
  async postVisit(page, context) {
    // Run accessibility tests
    await checkA11y(page, '#storybook-root', {
      detailedReport: true,
      detailedReportOptions: { html: true },
    });
  },
  tags: {
    include: ['visual-test'],
    exclude: ['skip-test'],
  },
};

export default config;
```

### Component Testing with Storybook Test Runner

```tsx
// src/components/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { expect, userEvent, within } from '@storybook/test';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    testRunner: {
      tags: ['visual-test'],
    },
  },
  argTypes: {
    variant: {
      control: { type: 'select' },
      options: ['primary', 'secondary', 'danger'],
    },
    size: {
      control: { type: 'select' },
      options: ['sm', 'md', 'lg'],
    },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    children: 'Primary Button',
    variant: 'primary',
  },
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button');
    
    // Test button interaction
    await userEvent.click(button);
    
    // Verify button state
    expect(button).toHaveClass('btn-primary');
    expect(button).toHaveTextContent('Primary Button');
  },
};

export const Loading: Story = {
  args: {
    children: 'Loading Button',
    loading: true,
  },
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    const button = canvas.getByRole('button');
    
    // Test loading state
    expect(button).toBeDisabled();
    expect(canvas.getByText(/loading/i)).toBeInTheDocument();
  },
};

// Visual regression test
export const VisualRegression: Story = {
  args: {
    children: 'Visual Test Button',
    variant: 'primary',
    size: 'lg',
  },
  parameters: {
    chromatic: {
      // Disable animations for consistent screenshots
      pauseAnimationAtEnd: true,
    },
  },
};
```

### Design System Integration

```typescript
// .storybook/design-tokens.ts
export const designTokens = {
  colors: {
    primary: {
      50: '#eff6ff',
      100: '#dbeafe',
      500: '#3b82f6',
      900: '#1e3a8a',
    },
    semantic: {
      success: '#10b981',
      warning: '#f59e0b',
      error: '#ef4444',
    },
  },
  spacing: {
    xs: '0.25rem',
    sm: '0.5rem',
    md: '1rem',
    lg: '1.5rem',
    xl: '2rem',
  },
  typography: {
    fontFamily: {
      sans: ['Inter', 'system-ui', 'sans-serif'],
      mono: ['JetBrains Mono', 'monospace'],
    },
    fontSize: {
      xs: '0.75rem',
      sm: '0.875rem',
      base: '1rem',
      lg: '1.125rem',
      xl: '1.25rem',
    },
  },
};
```

### Advanced Storybook Patterns

```tsx
// src/components/DataTable.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { DataTable } from './DataTable';
import { mockUsers } from '../mocks/users';

const meta: Meta<typeof DataTable> = {
  title: 'Components/DataTable',
  component: DataTable,
  parameters: {
    layout: 'fullscreen',
    docs: {
      description: {
        component: 'A responsive data table with sorting, filtering, and pagination.',
      },
    },
  },
  argTypes: {
    data: { control: false },
    onRowClick: { action: 'row-clicked' },
    onSort: { action: 'sorted' },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    data: mockUsers.slice(0, 10),
    columns: [
      { key: 'id', label: 'ID', sortable: true },
      { key: 'name', label: 'Name', sortable: true },
      { key: 'email', label: 'Email', sortable: true },
      { key: 'role', label: 'Role', sortable: true },
    ],
  },
};

export const WithPagination: Story = {
  args: {
    ...Default.args,
    pagination: {
      pageSize: 5,
      totalItems: 100,
    },
  },
};

export const WithFiltering: Story = {
  args: {
    ...Default.args,
    filters: [
      {
        key: 'role',
        label: 'Role',
        type: 'select',
        options: [
          { value: 'admin', label: 'Admin' },
          { value: 'user', label: 'User' },
          { value: 'guest', label: 'Guest' },
        ],
      },
    ],
  },
};

// Interactive story with complex interactions
export const InteractiveTable: Story = {
  play: async ({ canvasElement, args }) => {
    const canvas = within(canvasElement);
    
    // Test sorting
    const nameHeader = canvas.getByRole('columnheader', { name: /name/i });
    await userEvent.click(nameHeader);
    
    // Test filtering
    const roleFilter = canvas.getByRole('combobox', { name: /role/i });
    await userEvent.selectOptions(roleFilter, 'admin');
    
    // Test pagination
    const nextPageButton = canvas.getByRole('button', { name: /next page/i });
    await userEvent.click(nextPageButton);
    
    // Test row selection
    const firstRow = canvas.getByRole('row', { name: /john doe/i });
    await userEvent.click(firstRow);
    
    // Verify selection state
    expect(firstRow).toHaveAttribute('aria-selected', 'true');
  },
};
```

### Storybook with MSW Integration

```typescript
// .storybook/preview.ts
import type { Preview } from '@storybook/react';
import { initialize, mswDecorator } from 'msw-storybook-addon';
import { handlers } from '../src/mocks/handlers';

// Initialize MSW
initialize({
  onUnhandledRequest: 'bypass',
});

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/,
      },
    },
    msw: {
      handlers,
    },
  },
  decorators: [
    mswDecorator,
    (Story) => (
      <div style={{ padding: '2rem' }}>
        <Story />
      </div>
    ),
  ],
};

export default preview;
```

### Performance Testing in Storybook

```tsx
// src/components/HeavyComponent.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { HeavyComponent } from './HeavyComponent';

const meta: Meta<typeof HeavyComponent> = {
  title: 'Performance/HeavyComponent',
  component: HeavyComponent,
  parameters: {
    // Performance testing parameters
    chromatic: {
      delay: 2000, // Wait for animations to complete
    },
    testRunner: {
      tags: ['performance-test'],
    },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const WithLargeDataset: Story = {
  args: {
    items: Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      name: `Item ${i}`,
      value: Math.random() * 100,
    })),
  },
  parameters: {
    // Performance budgets
    performance: {
      budgets: [
        {
          metric: 'first-contentful-paint',
          threshold: 1000,
        },
        {
          metric: 'largest-contentful-paint',
          threshold: 2500,
        },
      ],
    },
  },
};
```

## 🎪 Next.js 15+ Testing Strategies

### Testing React 19 Actions

```tsx
// Testing React 19 Actions with useActionState
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { useActionState } from 'react';
import { UserForm } from './UserForm';

// Mock the action
const mockSubmitUserData = jest.fn();

describe('UserForm with React 19 Actions', () => {
  beforeEach(() => {
    mockSubmitUserData.mockClear();
  });

  it('should handle form submission with loading state', async () => {
    mockSubmitUserData.mockResolvedValue({
      message: 'User created successfully!',
      errors: {},
      isSubmitting: false
    });

    render(<UserForm />);
    
    const nameInput = screen.getByLabelText(/name/i);
    const emailInput = screen.getByLabelText(/email/i);
    const submitButton = screen.getByRole('button', { name: /create user/i });
    
    fireEvent.change(nameInput, { target: { value: 'John Doe' } });
    fireEvent.change(emailInput, { target: { value: 'john@example.com' } });
    
    fireEvent.click(submitButton);
    
    // Check loading state
    expect(screen.getByText('Creating...')).toBeInTheDocument();
    expect(submitButton).toBeDisabled();
    
    // Wait for completion
    await waitFor(() => {
      expect(screen.getByText('User created successfully!')).toBeInTheDocument();
    });
    
    expect(mockSubmitUserData).toHaveBeenCalledWith(
      expect.any(FormData)
    );
  });

  it('should display validation errors', async () => {
    mockSubmitUserData.mockResolvedValue({
      message: '',
      errors: { 
        name: 'Name is required', 
        email: 'Email is required' 
      },
      isSubmitting: false
    });

    render(<UserForm />);
    
    const submitButton = screen.getByRole('button', { name: /create user/i });
    fireEvent.click(submitButton);
    
    await waitFor(() => {
      expect(screen.getByText('Name is required')).toBeInTheDocument();
      expect(screen.getByText('Email is required')).toBeInTheDocument();
    });
  });
});
```

### Testing Optimistic Updates

```tsx
// Testing useOptimistic hook
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { TodoList } from './TodoList';

describe('TodoList with Optimistic Updates', () => {
  const mockTodos = [
    { id: '1', text: 'Existing todo', completed: false }
  ];

  it('should show optimistic updates immediately', async () => {
    const mockAddTodo = jest.fn();
    
    render(<TodoList todos={mockTodos} onAddTodo={mockAddTodo} />);
    
    const input = screen.getByPlaceholderText(/add a todo/i);
    const addButton = screen.getByRole('button', { name: /add todo/i });
    
    fireEvent.change(input, { target: { value: 'New todo' } });
    fireEvent.click(addButton);
    
    // Should show optimistic update immediately
    expect(screen.getByText('New todo')).toBeInTheDocument();
    expect(screen.getByText('(Saving...)')).toBeInTheDocument();
    
    // Wait for API call to complete
    await waitFor(() => {
      expect(screen.queryByText('(Saving...)')).not.toBeInTheDocument();
    });
  });
});
```

### Testing Next.js 15 with Turbopack

```typescript
// vitest.config.ts for Next.js 15 with Turbopack
import { defineConfig } from 'vitest/config';
import { resolve } from 'path';

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./test/setup.ts'],
    globals: true,
  },
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@/components': resolve(__dirname, './src/components'),
      '@/lib': resolve(__dirname, './src/lib'),
    },
  },
  // Support for Next.js 15 Turbopack
  esbuild: {
    target: 'node18',
  },
  // Enhanced coverage for React 19 features
  coverage: {
    provider: 'v8',
    reporter: ['text', 'json', 'html'],
    exclude: [
      'node_modules/',
      'test/',
      '**/*.d.ts',
      '**/*.config.*',
      '**/coverage/**',
    ],
    thresholds: {
      global: {
        branches: 85,
        functions: 85,
        lines: 85,
        statements: 85,
      },
    },
  },
});
```

### Testing Custom Elements with React 19

```tsx
// Testing custom elements integration
import { render, screen, fireEvent } from '@testing-library/react';
import { WebComponentIntegration } from './WebComponentIntegration';

// Mock custom element
class MockMyCounter extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `
      <button id="increment">+</button>
      <span id="value">0</span>
      <button id="decrement">-</button>
    `;
    this.value = 0;
    this.setupEventListeners();
  }
  
  setupEventListeners() {
    this.querySelector('#increment')?.addEventListener('click', () => {
      this.value++;
      this.dispatchEvent(new CustomEvent('increment', {
        detail: { value: this.value }
      }));
    });
  }
  
  set value(newValue) {
    this._value = newValue;
    this.querySelector('#value')!.textContent = newValue.toString();
  }
  
  get value() {
    return this._value;
  }
}

customElements.define('my-counter', MockMyCounter);

describe('WebComponentIntegration', () => {
  it('should interact with custom elements', () => {
    render(<WebComponentIntegration />);
    
    const counter = screen.getByRole('button', { name: '+' });
    fireEvent.click(counter);
    
    // Custom element should update
    expect(screen.getByText('1')).toBeInTheDocument();
  });
});
```

### App Router Testing Patterns

```typescript
// __tests__/app/products/[id]/page.test.tsx
import { render, screen } from '@testing-library/react';
import { notFound } from 'next/navigation';
import ProductPage from '@/app/products/[id]/page';
import { server } from '@/mocks/server';
import { http, HttpResponse } from 'msw';

// Mock Next.js navigation
vi.mock('next/navigation', () => ({
  notFound: vi.fn(),
  useRouter: () => ({
    push: vi.fn(),
    back: vi.fn(),
    forward: vi.fn(),
    refresh: vi.fn(),
  }),
  useParams: () => ({ id: 'product-1' }),
  useSearchParams: () => new URLSearchParams(),
}));

describe('ProductPage (App Router)', () => {
  test('should render product details for valid ID', async () => {
    const ProductPageComponent = await ProductPage({ params: { id: 'product-1' } });
    
    render(ProductPageComponent);

    await waitFor(() => {
      expect(screen.getByRole('heading', { level: 1 })).toHaveTextContent('Wireless Headphones');
      expect(screen.getByText('$199')).toBeInTheDocument();
      expect(screen.getByRole('button', { name: /add to cart/i })).toBeInTheDocument();
    });
  });

  test('should call notFound for invalid product ID', async () => {
    server.use(
      http.get('/api/products/:id', ({ params }) => {
        if (params.id === 'invalid-id') {
          return HttpResponse.json(
            { error: 'Product not found' },
            { status: 404 }
          );
        }
      })
    );

    await ProductPage({ params: { id: 'invalid-id' } });

    expect(notFound).toHaveBeenCalled();
  });

  test('should handle server-side data fetching', async () => {
    // Mock fetch for server components
    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: true,
      json: async () => ({
        id: 'product-1',
        name: 'Server-fetched Product',
        price: 299,
      }),
    });

    const ProductPageComponent = await ProductPage({ params: { id: 'product-1' } });
    
    render(ProductPageComponent);

    expect(screen.getByText('Server-fetched Product')).toBeInTheDocument();
    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining('/api/products/product-1'),
      expect.any(Object)
    );
  });
});
```

### Server Components Testing

```typescript
// __tests__/app/dashboard/ServerStatsComponent.test.tsx
import { render, screen } from '@testing-library/react';
import { ServerStatsComponent } from '@/app/dashboard/ServerStatsComponent';

// Test Server Components with mocked data
describe('ServerStatsComponent', () => {
  test('should render server-fetched statistics', async () => {
    // Mock the async server component
    const mockStats = {
      totalUsers: 1250,
      totalOrders: 3420,
      revenue: 125000,
      growth: 12.5,
    };

    global.fetch = vi.fn().mockResolvedValueOnce({
      ok: true,
      json: async () => mockStats,
    });

    // Server components are async functions
    const ComponentWithData = await ServerStatsComponent();
    
    render(ComponentWithData);

    expect(screen.getByText('1,250')).toBeInTheDocument();
    expect(screen.getByText('3,420')).toBeInTheDocument();
    expect(screen.getByText('$125,000')).toBeInTheDocument();
    expect(screen.getByText('+12.5%')).toBeInTheDocument();
  });

  test('should handle server-side errors gracefully', async () => {
    global.fetch = vi.fn().mockRejectedValueOnce(new Error('Database connection failed'));

    const ComponentWithError = await ServerStatsComponent();
    
    render(ComponentWithError);

    expect(screen.getByText(/failed to load statistics/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /retry/i })).toBeInTheDocument();
  });
});
```

### App Router Layout Testing

```typescript
// __tests__/app/layout.test.tsx
import { render, screen } from '@testing-library/react';
import RootLayout from '@/app/layout';
import { AuthProvider } from '@/contexts/AuthProvider';

describe('RootLayout', () => {
  test('should render layout with navigation and children', () => {
    const TestChild = () => <div>Test Page Content</div>;
    
    render(
      <RootLayout>
        <TestChild />
      </RootLayout>
    );

    // Check layout structure
    expect(screen.getByRole('banner')).toBeInTheDocument(); // Header
    expect(screen.getByRole('navigation')).toBeInTheDocument(); // Nav
    expect(screen.getByRole('main')).toBeInTheDocument(); // Main content
    expect(screen.getByRole('contentinfo')).toBeInTheDocument(); // Footer
    
    // Check child content
    expect(screen.getByText('Test Page Content')).toBeInTheDocument();
  });

  test('should provide authentication context', () => {
    const TestChild = () => {
      const { user, isAuthenticated } = useAuth();
      return (
        <div>
          <span data-testid="auth-status">
            {isAuthenticated ? `Logged in as ${user?.name}` : 'Not logged in'}
          </span>
        </div>
      );
    };
    
    render(
      <RootLayout>
        <TestChild />
      </RootLayout>
    );

    expect(screen.getByTestId('auth-status')).toHaveTextContent('Not logged in');
  });

  test('should handle theme switching', async () => {
    const user = userEvent.setup();
    
    const TestChild = () => <div>Content</div>;
    
    render(
      <RootLayout>
        <TestChild />
      </RootLayout>
    );

    const themeToggle = screen.getByRole('button', { name: /toggle theme/i });
    
    // Should start with light theme
    expect(document.documentElement).toHaveAttribute('data-theme', 'light');
    
    // Toggle to dark theme
    await user.click(themeToggle);
    expect(document.documentElement).toHaveAttribute('data-theme', 'dark');
  });
});
```

## 🎭 Playwright: Modern E2E Testing Excellence

### Enterprise Playwright Configuration

```typescript
// playwright.config.ts - Production-ready configuration
import { defineConfig, devices } from '@playwright/test';
import dotenv from 'dotenv';

// Load environment variables
dotenv.config({ path: '.env.test' });

export default defineConfig({
  testDir: './e2e',
  outputDir: './test-results',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 2 : undefined,
  
  // Enhanced reporting
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/results.xml' }],
    ['github'], // GitHub Actions integration
    ...(process.env.CI ? [['blob']] : []), // Blob reporter for CI
  ],
  
  // Global test configuration
  use: {
    baseURL: process.env.E2E_BASE_URL || 'http://localhost:3000',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    
    // Modern browser features
    permissions: ['geolocation', 'camera', 'microphone'],
    geolocation: { longitude: -122.4194, latitude: 37.7749 }, // San Francisco
    locale: 'en-US',
    timezoneId: 'America/New_York',
    
    // Performance and accessibility
    extraHTTPHeaders: {
      'Accept-Language': 'en-US,en;q=0.9',
    },
  },

  // Test projects for different browsers and scenarios
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
      name: 'mobile-chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'mobile-safari',
      use: { ...devices['iPhone 12'] },
    },
    
    // Tablet testing
    {
      name: 'tablet',
      use: { ...devices['iPad Pro'] },
    },
    
    // Accessibility testing
    {
      name: 'accessibility',
      use: { ...devices['Desktop Chrome'] },
      testMatch: /.*\.a11y\.spec\.ts/,
    },
    
    // Performance testing
    {
      name: 'performance',
      use: { 
        ...devices['Desktop Chrome'],
        // Simulate slower connection
        extraHTTPHeaders: {
          'User-Agent': 'Mozilla/5.0 (compatible; Performance Test)',
        },
      },
      testMatch: /.*\.perf\.spec\.ts/,
    },
    
    // Visual regression testing
    {
      name: 'visual',
      use: { ...devices['Desktop Chrome'] },
      testMatch: /.*\.visual\.spec\.ts/,
    },
  ],

  // Development server
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
    env: {
      NODE_ENV: 'test',
    },
  },

  // Global setup and teardown
  globalSetup: require.resolve('./e2e/global-setup'),
  globalTeardown: require.resolve('./e2e/global-teardown'),
});
```

### Advanced Page Object Models

```typescript
// e2e/pages/BasePage.ts - Base page with common functionality
import { Page, Locator, expect } from '@playwright/test';

export abstract class BasePage {
  protected page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Navigation helpers
  async goto(path: string = '') {
    await this.page.goto(path);
    await this.waitForLoadState();
  }

  async waitForLoadState() {
    await this.page.waitForLoadState('networkidle');
    await this.page.waitForLoadState('domcontentloaded');
  }

  // Accessibility helpers
  async checkAccessibility() {
    const { axe } = await import('@axe-core/playwright');
    const results = await axe(this.page);
    expect(results.violations).toEqual([]);
  }

  // Performance helpers
  async measurePerformance() {
    return await this.page.evaluate(() => {
      const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
      const paint = performance.getEntriesByType('paint');
      
      return {
        domContentLoaded: navigation.domContentLoadedEventEnd - navigation.navigationStart,
        loadComplete: navigation.loadEventEnd - navigation.navigationStart,
        firstPaint: paint.find(p => p.name === 'first-paint')?.startTime || 0,
        firstContentfulPaint: paint.find(p => p.name === 'first-contentful-paint')?.startTime || 0,
        timeToInteractive: navigation.domInteractive - navigation.navigationStart,
      };
    });
  }

  // Core Web Vitals measurement
  async measureCoreWebVitals() {
    return await this.page.evaluate(() => {
      return new Promise((resolve) => {
        const vitals = {
          lcp: 0,
          fid: 0,
          cls: 0,
          inp: 0, // New metric
        };

        // LCP Observer
        new PerformanceObserver((list) => {
          const entries = list.getEntries();
          vitals.lcp = entries[entries.length - 1].startTime;
        }).observe({ entryTypes: ['largest-contentful-paint'] });

        // CLS Observer
        let clsValue = 0;
        new PerformanceObserver((list) => {
          list.getEntries().forEach((entry: any) => {
            if (!entry.hadRecentInput) {
              clsValue += entry.value;
              vitals.cls = clsValue;
            }
          });
        }).observe({ entryTypes: ['layout-shift'] });

        // INP Observer (replacing FID)
        new PerformanceObserver((list) => {
          list.getEntries().forEach((entry: any) => {
            if (entry.processingStart && entry.startTime) {
              const inp = entry.processingStart - entry.startTime;
              vitals.inp = Math.max(vitals.inp, inp);
            }
          });
        }).observe({ entryTypes: ['event'] });

        // Return values after a delay to collect metrics
        setTimeout(() => resolve(vitals), 2000);
      });
    });
  }

  // Wait for specific elements with timeout
  async waitForElement(locator: Locator, timeout: number = 10000) {
    await locator.waitFor({ state: 'visible', timeout });
  }

  // Screenshot helpers
  async takeScreenshot(name: string) {
    await this.page.screenshot({
      path: `screenshots/${name}.png`,
      fullPage: true,
    });
  }

  // Form helpers
  async fillForm(formData: Record<string, string>) {
    for (const [field, value] of Object.entries(formData)) {
      const input = this.page.getByLabel(new RegExp(field, 'i'));
      await input.fill(value);
    }
  }

  // Cookie and localStorage helpers
  async clearStorage() {
    await this.page.evaluate(() => {
      localStorage.clear();
      sessionStorage.clear();
    });
    await this.page.context().clearCookies();
  }

  async setAuthToken(token: string) {
    await this.page.evaluate((token) => {
      localStorage.setItem('authToken', token);
    }, token);
  }

  // Error handling
  async handleErrors() {
    const errors: string[] = [];
    
    this.page.on('pageerror', (error) => {
      errors.push(error.message);
    });
    
    this.page.on('requestfailed', (request) => {
      errors.push(`Request failed: ${request.url()}`);
    });
    
    return errors;
  }
}
```

### Advanced E2E Test Patterns

```typescript
// e2e/specs/e-commerce-flow.spec.ts
import { test, expect } from '@playwright/test';
import { HomePage } from '../pages/HomePage';
import { ProductPage } from '../pages/ProductPage';
import { CartPage } from '../pages/CartPage';
import { CheckoutPage } from '../pages/CheckoutPage';
import { AuthPage } from '../pages/AuthPage';

test.describe('E-Commerce User Journey', () => {
  let homePage: HomePage;
  let productPage: ProductPage;
  let cartPage: CartPage;
  let checkoutPage: CheckoutPage;
  let authPage: AuthPage;

  test.beforeEach(async ({ page }) => {
    homePage = new HomePage(page);
    productPage = new ProductPage(page);
    cartPage = new CartPage(page);
    checkoutPage = new CheckoutPage(page);
    authPage = new AuthPage(page);
    
    await homePage.goto();
  });

  test('should complete full purchase journey with performance checks', async ({ page }) => {
    // Performance baseline
    const initialMetrics = await homePage.measurePerformance();
    expect(initialMetrics.domContentLoaded).toBeLessThan(2000);
    
    // Search and select product
    await homePage.searchProducts('wireless headphones');
    await homePage.selectProduct('Wireless Headphones Pro');
    
    // Verify product details
    await productPage.waitForProductLoad();
    await expect(productPage.productTitle).toContainText('Wireless Headphones Pro');
    await expect(productPage.price).toBeVisible();
    
    // Add to cart
    await productPage.addToCart();
    await expect(productPage.successMessage).toContainText('Added to cart');
    
    // Verify cart
    await productPage.goToCart();
    await cartPage.waitForCartLoad();
    await expect(cartPage.cartItems).toHaveCount(1);
    
    // Proceed to checkout (requires authentication)
    await cartPage.proceedToCheckout();
    
    // Handle authentication
    if (await authPage.isLoginRequired()) {
      await authPage.login('test@example.com', 'password123');
      await authPage.waitForRedirect();
    }
    
    // Complete checkout
    await checkoutPage.waitForCheckoutLoad();
    await checkoutPage.fillShippingInfo({
      firstName: 'John',
      lastName: 'Doe',
      address: '123 Test St',
      city: 'Test City',
      state: 'CA',
      zipCode: '12345',
    });
    
    await checkoutPage.fillPaymentInfo({
      cardNumber: '4111111111111111',
      expiryDate: '12/25',
      cvv: '123',
    });
    
    await checkoutPage.placeOrder();
    
    // Verify order confirmation
    await expect(page.getByText(/order confirmed/i)).toBeVisible();
    await expect(page.getByText(/order #/i)).toBeVisible();
    
    // Performance check after completion
    const finalMetrics = await homePage.measureCoreWebVitals();
    expect(finalMetrics.lcp).toBeLessThan(2500); // LCP < 2.5s
    expect(finalMetrics.inp).toBeLessThan(200);  // INP < 200ms
    expect(finalMetrics.cls).toBeLessThan(0.1);  // CLS < 0.1
  });

  test('should handle cart persistence across sessions', async ({ context, page }) => {
    // Add item to cart
    await homePage.searchProducts('laptop');
    await homePage.selectProduct('Gaming Laptop');
    await productPage.addToCart();
    
    // Create new page (simulate new session)
    const newPage = await context.newPage();
    const newHomePage = new HomePage(newPage);
    await newHomePage.goto();
    
    // Verify cart persistence
    const cartCount = await newHomePage.getCartCount();
    expect(cartCount).toBe('1');
    
    // Verify cart contents
    await newHomePage.goToCart();
    const newCartPage = new CartPage(newPage);
    await newCartPage.waitForCartLoad();
    await expect(newCartPage.cartItems).toHaveCount(1);
  });

  test('should handle network failures gracefully', async ({ page }) => {
    // Add product to cart
    await homePage.searchProducts('headphones');
    await homePage.selectProduct('Wireless Headphones');
    await productPage.addToCart();
    await productPage.goToCart();
    
    // Simulate network failure
    await page.route('**/api/orders', route => route.abort('failed'));
    
    await cartPage.proceedToCheckout();
    
    // Skip auth for this test
    await page.route('**/api/auth/check', route => 
      route.fulfill({
        status: 200,
        body: JSON.stringify({ authenticated: true }),
      })
    );
    
    await checkoutPage.fillShippingInfo({
      firstName: 'John',
      lastName: 'Doe',
      address: '123 Test St',
      city: 'Test City',
      state: 'CA',
      zipCode: '12345',
    });
    
    await checkoutPage.placeOrder();
    
    // Should show error message
    await expect(page.getByText(/network error/i)).toBeVisible();
    await expect(page.getByRole('button', { name: /try again/i })).toBeVisible();
  });

  test('should support mobile checkout flow', async ({ page }) => {
    // Set mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    
    // Mobile-specific interactions
    await homePage.openMobileMenu();
    await homePage.searchProducts('phone case');
    await homePage.selectProduct('Clear Phone Case');
    
    // Verify mobile product view
    await productPage.waitForProductLoad();
    expect(await productPage.isMobileLayout()).toBe(true);
    
    await productPage.addToCart();
    
    // Mobile cart interaction
    await productPage.openMobileCart();
    await cartPage.proceedToCheckoutMobile();
    
    // Mobile checkout form
    await checkoutPage.fillMobileCheckoutForm({
      email: 'mobile@test.com',
      phone: '+1234567890',
      address: '456 Mobile St',
    });
    
    await checkoutPage.placeOrderMobile();
    
    // Verify mobile success page
    await expect(page.getByTestId('mobile-success-message')).toBeVisible();
  });

  test('should maintain performance under load', async ({ page }) => {
    // Simulate multiple concurrent users
    const promises = [];
    
    for (let i = 0; i < 5; i++) {
      promises.push(
        (async () => {
          await homePage.searchProducts(`product ${i}`);
          await page.waitForTimeout(100); // Small delay between actions
        })()
      );
    }
    
    await Promise.all(promises);
    
    // Measure performance impact
    const metrics = await homePage.measurePerformance();
    expect(metrics.domContentLoaded).toBeLessThan(3000); // Allow slight degradation
    
    // Check for JavaScript errors
    const errors = await homePage.handleErrors();
    expect(errors).toHaveLength(0);
  });

  test('should support accessibility throughout the journey', async ({ page }) => {
    // Check homepage accessibility
    await homePage.checkAccessibility();
    
    // Search with keyboard navigation
    await page.keyboard.press('Tab'); // Focus search
    await page.keyboard.type('bluetooth speaker');
    await page.keyboard.press('Enter');
    
    // Navigate search results with keyboard
    await page.keyboard.press('Tab');
    await page.keyboard.press('Tab');
    await page.keyboard.press('Enter'); // Select product
    
    // Check product page accessibility
    await productPage.waitForProductLoad();
    await productPage.checkAccessibility();
    
    // Add to cart using keyboard
    await page.keyboard.press('Tab');
    await page.keyboard.press('Tab');
    await page.keyboard.press('Enter'); // Add to cart button
    
    // Check cart accessibility
    await productPage.goToCart();
    await cartPage.checkAccessibility();
    
    // Verify screen reader announcements
    const announcements = await page.getByRole('status').allTextContents();
    expect(announcements.some(text => text.includes('added to cart'))).toBe(true);
  });
});
```

## 📋 Comprehensive Testing Exercises

### Exercise 1: Modern Testing Stack Implementation

**Objective**: Set up a complete testing infrastructure for a React/Next.js application.

**Requirements**:
1. **Testing Architecture**: Implement Testing Trophy distribution (40% Static, 50% Integration, 20% Unit, 10% E2E)
2. **Tool Selection**: Configure Vitest, Playwright, MSW 2.0, and accessibility testing
3. **AI Integration**: Set up at least one AI-powered testing tool (Applitools, TestRigor)
4. **Performance Testing**: Implement Core Web Vitals testing in your test suite
5. **Contract Testing**: Set up Pact for API contract testing

**Deliverables**:
- Complete Vitest configuration with advanced features
- MSW 2.0 setup with realistic API mocking
- Playwright configuration for cross-browser testing
- CI/CD pipeline with parallel test execution
- Performance and accessibility testing integration
- Documentation of testing strategy and tool choices

### Exercise 2: AI-Powered Testing Implementation

**Objective**: Integrate AI-powered testing tools into an existing test suite.

**Tasks**:
1. Set up Applitools for visual AI testing
2. Implement TestRigor natural language tests for user workflows
3. Configure mutation testing with Stryker and AI analysis
4. Create property-based tests with fast-check for edge case discovery
5. Set up AI-powered test generation and maintenance

**Success Criteria**:
- AI tools successfully catch visual regressions
- Natural language tests cover critical user journeys
- Mutation testing achieves >85% mutation score
- Property-based tests discover previously unknown edge cases
- AI-generated tests provide meaningful coverage improvements

### Exercise 3: Next.js 15+ Testing Mastery

**Objective**: Master testing patterns for modern Next.js 15 applications with App Router and React 19 features.

**Implementation Areas**:
1. Server Components testing strategies
2. App Router page and layout testing
3. Server Actions testing
4. Streaming and Suspense testing
5. Middleware testing
6. API Routes testing with contract validation

**Expected Outcomes**:
- Comprehensive test coverage for App Router features
- Effective Server Components testing approach
- Performance testing integration for SSR/RSC
- Contract testing for API routes
- Security testing for server-side functionality

### Exercise 4: Enterprise-Grade E2E Testing

**Objective**: Build a production-ready E2E testing suite with Playwright.

**Scope**:
1. Advanced Page Object Models with inheritance
2. Cross-browser and mobile testing
3. Performance testing with Core Web Vitals
4. Accessibility testing automation
5. Visual regression testing with AI
6. API testing integration
7. Parallel execution and test optimization

**Validation**:
- Tests run reliably across all major browsers
- Performance budgets enforced in E2E tests
- Full accessibility compliance verification
- Visual changes automatically detected and reviewed
- Test execution time optimized for CI/CD

### Exercise 5: Micro-Frontend Testing Strategy

**Objective**: Develop comprehensive testing for micro-frontend architecture.

**Components**:
1. Individual micro-frontend unit and integration testing
2. Cross-micro-frontend integration testing
3. Contract testing between micro-frontends
4. Performance testing for federated modules
5. End-to-end testing across the entire system
6. Deployment safety testing

**Outcomes**:
- Each micro-frontend has independent test suite
- Integration contracts prevent breaking changes
- Performance impact of federation measured
- Complete user journeys tested across boundaries
- Safe deployment strategies validated through testing

### Final Project: Complete Testing Excellence Implementation

**Objective**: Create a comprehensive testing strategy that demonstrates mastery of all testing practices.

**Requirements**:
1. **Architecture**: Implement Testing Trophy with metrics tracking
2. **Tools**: Use modern stack (Vitest, Playwright, MSW 2.0, AI tools)
3. **Coverage**: Achieve >85% branch coverage with meaningful tests
4. **Performance**: Integrate Core Web Vitals testing and budgets
5. **Accessibility**: Automate WCAG 2.1 AA compliance checking
6. **Security**: Include XSS, CSP, and vulnerability testing
7. **CI/CD**: Implement parallel execution with intelligent test selection
8. **Observability**: Set up comprehensive testing metrics and reporting

**Deliverable**: A production-ready application with enterprise-grade testing demonstrating:
- Complete test pyramid implementation
- AI-powered testing capabilities  
- Performance and accessibility compliance
- Security testing integration
- Advanced E2E testing with cross-browser support
- Comprehensive documentation and team training materials

---

**Next:** [Module 5: Production-Grade Code Practices](../05-code-practices/) - Master SOLID principles, error handling, performance optimization, and enterprise architecture patterns.