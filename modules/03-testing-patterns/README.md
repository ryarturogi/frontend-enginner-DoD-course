# Module 3: Testing Patterns for Frontend

## 🎯 Learning Objectives

By the end of this module, you will:
- Master the testing pyramid and understand when to use each testing level
- Apply AAA, GWT, SEAT, and Four-Phase testing patterns effectively
- Implement property-based testing for robust test coverage
- Use Golden Master testing for legacy code and UI components
- Apply BDD principles with Cucumber/Gherkin for business-readable tests
- Choose the right testing pattern for different scenarios

## 🔺 The Testing Pyramid

The testing pyramid is your guide for balancing test coverage, speed, and confidence.

```
       E2E Tests
      (High Cost)
     /           \
    /             \
   /               \
  /  Integration    \
 /     Tests         \
/     (Medium)        \
_____________________
     Unit Tests
    (Low Cost)
```

### Unit Tests (70% of tests)
- **What:** Test individual functions, components, or modules in isolation
- **Speed:** Very fast (milliseconds)
- **Cost:** Low to write and maintain
- **Confidence:** High for logic, low for integration
- **Tools:** Jest, Vitest, React Testing Library

### Integration Tests (20% of tests)
- **What:** Test how multiple components work together
- **Speed:** Medium (seconds)
- **Cost:** Medium to write and maintain
- **Confidence:** High for user workflows
- **Tools:** React Testing Library, Testing Library variants

### E2E Tests (10% of tests)
- **What:** Test complete user journeys in real browser
- **Speed:** Slow (minutes)
- **Cost:** High to write and maintain
- **Confidence:** Highest for real user scenarios
- **Tools:** Playwright, Cypress

### Why This Balance?

- **Speed:** Unit tests give fast feedback during development
- **Cost:** E2E tests are expensive to write and maintain
- **Reliability:** Unit tests are more stable and less flaky
- **Coverage:** Unit tests can cover edge cases easily
- **Confidence:** E2E tests catch real integration issues

## 🧪 Testing Patterns Deep Dive

### 1. AAA Pattern (Arrange-Act-Assert)

The most fundamental testing pattern for clear, readable tests.

#### Structure
```javascript
// Arrange: Set up test data and conditions
// Act: Execute the behavior being tested  
// Assert: Verify the expected outcome
```

#### Example: Testing a Calculator Function
```javascript
// ❌ Poor structure
test('calculator works', () => {
  expect(add(2, 3)).toBe(5);
  expect(multiply(2, 3)).toBe(6);
  expect(subtract(5, 2)).toBe(3);
});

// ✅ Good AAA structure
describe('Calculator', () => {
  test('should add two positive numbers', () => {
    // Arrange
    const firstNumber = 2;
    const secondNumber = 3;
    const expectedResult = 5;
    
    // Act
    const result = add(firstNumber, secondNumber);
    
    // Assert
    expect(result).toBe(expectedResult);
  });
  
  test('should handle negative numbers', () => {
    // Arrange
    const firstNumber = -2;
    const secondNumber = 3;
    const expectedResult = 1;
    
    // Act
    const result = add(firstNumber, secondNumber);
    
    // Assert
    expect(result).toBe(expectedResult);
  });
});
```

#### AAA for React Components
```javascript
describe('LoginForm', () => {
  test('should display error message for invalid credentials', async () => {
    // Arrange
    const mockOnSubmit = jest.fn().mockRejectedValue(
      new Error('Invalid credentials')
    );
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    const emailInput = screen.getByLabelText(/email/i);
    const passwordInput = screen.getByLabelText(/password/i);
    const submitButton = screen.getByRole('button', { name: /login/i });
    
    // Act
    await user.type(emailInput, 'test@example.com');
    await user.type(passwordInput, 'wrongpassword');
    await user.click(submitButton);
    
    // Assert
    expect(await screen.findByText(/invalid credentials/i)).toBeInTheDocument();
    expect(mockOnSubmit).toHaveBeenCalledWith({
      email: 'test@example.com',
      password: 'wrongpassword'
    });
  });
});
```

#### AAA Best Practices
- **Keep sections distinct** with blank lines
- **One assertion per test** when possible
- **Use descriptive variable names** in Arrange section
- **Make assertions specific** and meaningful

### 2. GWT Pattern (Given-When-Then)

Behavior-driven pattern that makes tests readable by non-technical stakeholders.

#### Structure
```javascript
// Given: The initial context and conditions
// When: The action or event that triggers the behavior
// Then: The expected outcome or result
```

#### Example: User Authentication Flow
```javascript
describe('User Authentication', () => {
  test('Given a user with valid credentials, When they attempt to login, Then they should be redirected to dashboard', async () => {
    // Given: A user with valid credentials
    const validUser = {
      email: 'user@example.com',
      password: 'validpassword123'
    };
    const mockAuthService = {
      login: jest.fn().mockResolvedValue({ 
        token: 'valid-jwt-token',
        user: { id: 1, email: validUser.email }
      })
    };
    const mockNavigate = jest.fn();
    
    render(
      <AuthProvider authService={mockAuthService}>
        <LoginPage navigate={mockNavigate} />
      </AuthProvider>
    );
    
    // When: They attempt to login
    await user.type(screen.getByLabelText(/email/i), validUser.email);
    await user.type(screen.getByLabelText(/password/i), validUser.password);
    await user.click(screen.getByRole('button', { name: /login/i }));
    
    // Then: They should be redirected to dashboard
    await waitFor(() => {
      expect(mockNavigate).toHaveBeenCalledWith('/dashboard');
    });
    expect(mockAuthService.login).toHaveBeenCalledWith(validUser);
  });
});
```

#### GWT for Business Logic
```javascript
describe('Shopping Cart', () => {
  test('Given an empty cart, When user adds a product, Then cart should contain one item', () => {
    // Given: An empty cart
    const cart = new ShoppingCart();
    const product = { id: 1, name: 'Laptop', price: 999.99 };
    
    expect(cart.getItemCount()).toBe(0);
    
    // When: User adds a product
    cart.addProduct(product);
    
    // Then: Cart should contain one item
    expect(cart.getItemCount()).toBe(1);
    expect(cart.getItems()).toContain(product);
    expect(cart.getTotal()).toBe(999.99);
  });
  
  test('Given a cart with duplicate products, When user adds same product, Then quantity should increase', () => {
    // Given: A cart with duplicate products
    const cart = new ShoppingCart();
    const product = { id: 1, name: 'Laptop', price: 999.99 };
    cart.addProduct(product);
    
    expect(cart.getProductQuantity(product.id)).toBe(1);
    
    // When: User adds same product
    cart.addProduct(product);
    
    // Then: Quantity should increase
    expect(cart.getProductQuantity(product.id)).toBe(2);
    expect(cart.getTotal()).toBe(1999.98);
  });
});
```

#### GWT vs AAA: When to Use Which?

**Use GWT when:**
- Tests will be read by non-technical stakeholders
- Writing behavior-driven tests
- Testing complex business logic
- Working with acceptance criteria

**Use AAA when:**
- Writing technical unit tests
- Testing implementation details
- Team is primarily technical
- Performance and brevity matter

### 3. SEAT Pattern (Setup-Execute-Assert-Teardown)

Comprehensive pattern for integration and E2E tests that require setup and cleanup.

#### Structure
```javascript
// Setup: Prepare test environment and data
// Execute: Run the action being tested
// Assert: Verify expected outcomes
// Teardown: Clean up resources and state
```

#### Example: Integration Test with Database
```javascript
describe('User Registration Integration', () => {
  let testDb;
  let server;
  let apiClient;
  
  beforeAll(async () => {
    // Setup: Initialize test environment
    testDb = await createTestDatabase();
    server = await startTestServer(testDb);
    apiClient = new ApiClient(server.url);
  });
  
  afterAll(async () => {
    // Teardown: Clean up environment
    await server.close();
    await testDb.close();
  });
  
  beforeEach(async () => {
    // Setup: Fresh state for each test
    await testDb.clear();
  });
  
  test('should create user and send welcome email', async () => {
    // Setup: Prepare test data
    const userData = {
      email: 'newuser@example.com',
      password: 'securepassword123',
      name: 'John Doe'
    };
    const mockEmailService = jest.spyOn(emailService, 'sendWelcomeEmail');
    
    // Execute: Register new user
    const response = await apiClient.post('/users/register', userData);
    
    // Assert: Verify user creation and email
    expect(response.status).toBe(201);
    expect(response.data.user.email).toBe(userData.email);
    expect(response.data.user.id).toBeDefined();
    
    const userInDb = await testDb.users.findByEmail(userData.email);
    expect(userInDb).toBeTruthy();
    expect(userInDb.name).toBe(userData.name);
    
    expect(mockEmailService).toHaveBeenCalledWith(userData.email, userData.name);
    
    // Teardown: Clean up mocks (handled by Jest automatically)
  });
});
```

#### SEAT for E2E Tests
```javascript
describe('E-commerce Checkout Flow', () => {
  let browser;
  let page;
  let testUser;
  let testProducts;
  
  beforeAll(async () => {
    // Setup: Launch browser and prepare test data
    browser = await playwright.chromium.launch();
    testUser = await createTestUser();
    testProducts = await createTestProducts();
  });
  
  afterAll(async () => {
    // Teardown: Clean up browser and test data
    await browser.close();
    await cleanupTestUser(testUser.id);
    await cleanupTestProducts(testProducts.map(p => p.id));
  });
  
  beforeEach(async () => {
    // Setup: Fresh page and login for each test
    page = await browser.newPage();
    await page.goto('/login');
    await loginUser(page, testUser);
  });
  
  afterEach(async () => {
    // Teardown: Close page and clear cart
    await page.close();
    await clearUserCart(testUser.id);
  });
  
  test('should complete purchase with credit card', async () => {
    // Setup: Add products to cart
    await page.goto('/products');
    await page.click(`[data-testid="add-to-cart-${testProducts[0].id}"]`);
    await page.click(`[data-testid="add-to-cart-${testProducts[1].id}"]`);
    
    // Execute: Go through checkout flow
    await page.click('[data-testid="cart-icon"]');
    await page.click('[data-testid="checkout-button"]');
    
    // Fill shipping information
    await page.fill('[data-testid="shipping-address"]', '123 Test St');
    await page.fill('[data-testid="shipping-city"]', 'Test City');
    await page.selectOption('[data-testid="shipping-state"]', 'CA');
    
    // Fill payment information
    await page.fill('[data-testid="card-number"]', '4111111111111111');
    await page.fill('[data-testid="card-expiry"]', '12/25');
    await page.fill('[data-testid="card-cvv"]', '123');
    
    await page.click('[data-testid="place-order-button"]');
    
    // Assert: Verify successful purchase
    await page.waitForSelector('[data-testid="order-confirmation"]');
    expect(await page.textContent('[data-testid="order-status"]')).toBe('Order Confirmed');
    
    const orderNumber = await page.textContent('[data-testid="order-number"]');
    expect(orderNumber).toMatch(/^ORD-\d+$/);
    
    // Verify in database
    const order = await testDb.orders.findByNumber(orderNumber);
    expect(order.status).toBe('confirmed');
    expect(order.items).toHaveLength(2);
    
    // Teardown is handled by afterEach/afterAll
  });
});
```

### 4. Four-Phase Test Pattern (xUnit Style)

Similar to SEAT but with more explicit phase separation, common in xUnit testing frameworks.

#### Structure
```javascript
// Phase 1: Setup - Create test fixtures and dependencies
// Phase 2: Exercise - Execute the system under test
// Phase 3: Verify - Check expected outcomes
// Phase 4: Teardown - Clean up test fixtures
```

#### Example: Component State Management Test
```javascript
describe('UserProfile Component State Management', () => {
  // Phase 1: Setup helpers
  const createUserProfileSetup = () => {
    const mockUser = {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      avatar: 'https://example.com/avatar.jpg'
    };
    
    const mockUpdateUser = jest.fn();
    const mockUploadAvatar = jest.fn();
    
    const renderComponent = () => render(
      <UserProfile 
        user={mockUser}
        onUpdateUser={mockUpdateUser}
        onUploadAvatar={mockUploadAvatar}
      />
    );
    
    return {
      mockUser,
      mockUpdateUser,
      mockUploadAvatar,
      renderComponent
    };
  };
  
  test('should update user name and trigger save', async () => {
    // Phase 1: Setup
    const { 
      mockUser, 
      mockUpdateUser, 
      renderComponent 
    } = createUserProfileSetup();
    
    renderComponent();
    
    const nameInput = screen.getByDisplayValue(mockUser.name);
    const saveButton = screen.getByRole('button', { name: /save/i });
    const newName = 'Jane Doe';
    
    // Phase 2: Exercise
    await user.clear(nameInput);
    await user.type(nameInput, newName);
    await user.click(saveButton);
    
    // Phase 3: Verify
    expect(mockUpdateUser).toHaveBeenCalledWith({
      ...mockUser,
      name: newName
    });
    expect(screen.getByDisplayValue(newName)).toBeInTheDocument();
    
    // Phase 4: Teardown (handled by testing framework)
  });
  
  test('should handle avatar upload with preview', async () => {
    // Phase 1: Setup
    const { 
      mockUploadAvatar, 
      renderComponent 
    } = createUserProfileSetup();
    
    renderComponent();
    
    const avatarInput = screen.getByLabelText(/upload avatar/i);
    const testFile = new File(['test'], 'avatar.jpg', { type: 'image/jpeg' });
    
    // Phase 2: Exercise
    await user.upload(avatarInput, testFile);
    
    // Phase 3: Verify
    expect(mockUploadAvatar).toHaveBeenCalledWith(testFile);
    
    // Verify preview appears
    const previewImage = await screen.findByAltText(/avatar preview/i);
    expect(previewImage).toBeInTheDocument();
    
    // Phase 4: Teardown (automatic cleanup)
  });
});
```

### 5. Property-Based Testing

Test with randomly generated inputs to discover edge cases you didn't think of.

#### Concept
Instead of testing specific examples, define properties that should always be true.

#### Example: Input Validation
```javascript
import fc from 'fast-check';

describe('Email Validation', () => {
  test('should accept valid email formats', () => {
    fc.assert(
      fc.property(
        fc.emailAddress(),
        (email) => {
          // Property: All generated email addresses should be valid
          expect(isValidEmail(email)).toBe(true);
        }
      )
    );
  });
  
  test('should reject strings without @ symbol', () => {
    fc.assert(
      fc.property(
        fc.string().filter(s => !s.includes('@')),
        (invalidEmail) => {
          // Property: Strings without @ should always be invalid
          expect(isValidEmail(invalidEmail)).toBe(false);
        }
      )
    );
  });
  
  test('should handle edge cases in password strength', () => {
    fc.assert(
      fc.property(
        fc.string({ minLength: 1, maxLength: 7 }),
        (shortPassword) => {
          // Property: Passwords under 8 characters should be weak
          const strength = calculatePasswordStrength(shortPassword);
          expect(strength).toBeLessThan(50);
        }
      )
    );
  });
});
```

#### Property-Based Testing for React Components
```javascript
describe('PriceDisplay Component', () => {
  test('should always display currency symbol for valid prices', () => {
    fc.assert(
      fc.property(
        fc.float({ min: 0, max: 999999.99 }),
        fc.constantFrom('USD', 'EUR', 'GBP'),
        (price, currency) => {
          render(<PriceDisplay price={price} currency={currency} />);
          
          const priceText = screen.getByTestId('price-display').textContent;
          
          // Property: Should always include currency symbol
          const currencySymbols = { USD: '$', EUR: '€', GBP: '£' };
          expect(priceText).toContain(currencySymbols[currency]);
          
          // Property: Should always include the price
          expect(priceText).toContain(price.toFixed(2));
        }
      )
    );
  });
  
  test('should handle extreme values gracefully', () => {
    fc.assert(
      fc.property(
        fc.oneof(
          fc.constant(0),
          fc.constant(Number.MAX_SAFE_INTEGER),
          fc.constant(Number.MIN_SAFE_INTEGER),
          fc.constant(Infinity),
          fc.constant(-Infinity),
          fc.constant(NaN)
        ),
        (extremeValue) => {
          // Property: Component should not crash with extreme values
          expect(() => {
            render(<PriceDisplay price={extremeValue} currency="USD" />);
          }).not.toThrow();
          
          const display = screen.getByTestId('price-display');
          expect(display).toBeInTheDocument();
        }
      )
    );
  });
});
```

### 6. Golden Master Testing (Approval Testing)

Perfect for testing complex outputs or when migrating legacy code.

#### Concept
1. Run the system and capture output
2. Review and approve the output as "golden master"
3. Future runs compare against the master
4. Changes require explicit approval

#### Example: Component Snapshot Testing
```javascript
describe('ProductCard Component Snapshots', () => {
  const sampleProduct = {
    id: 1,
    name: 'Wireless Headphones',
    price: 199.99,
    rating: 4.5,
    image: 'https://example.com/headphones.jpg',
    inStock: true
  };
  
  test('should match snapshot for standard product', () => {
    const { container } = render(<ProductCard product={sampleProduct} />);
    expect(container.firstChild).toMatchSnapshot();
  });
  
  test('should match snapshot for out-of-stock product', () => {
    const outOfStockProduct = { ...sampleProduct, inStock: false };
    const { container } = render(<ProductCard product={outOfStockProduct} />);
    expect(container.firstChild).toMatchSnapshot();
  });
  
  test('should match snapshot for product on sale', () => {
    const saleProduct = { 
      ...sampleProduct, 
      originalPrice: 249.99,
      salePrice: 199.99 
    };
    const { container } = render(<ProductCard product={saleProduct} />);
    expect(container.firstChild).toMatchSnapshot();
  });
});
```

#### Golden Master for Complex Business Logic
```javascript
describe('Shopping Cart Calculations', () => {
  const complexCartScenarios = [
    {
      name: 'multiple items with tax and shipping',
      cart: {
        items: [
          { id: 1, price: 29.99, quantity: 2, taxable: true },
          { id: 2, price: 49.99, quantity: 1, taxable: false },
          { id: 3, price: 15.99, quantity: 3, taxable: true }
        ],
        shipping: { method: 'standard', cost: 9.99 },
        tax: { rate: 0.0875, zipCode: '90210' },
        discounts: [
          { type: 'percentage', value: 10, code: 'SAVE10' }
        ]
      }
    },
    // ... more complex scenarios
  ];
  
  complexCartScenarios.forEach(scenario => {
    test(`should calculate correctly for ${scenario.name}`, () => {
      const calculator = new CartCalculator();
      const result = calculator.calculate(scenario.cart);
      
      // Golden master: Capture complete calculation breakdown
      expect(result).toMatchSnapshot({
        // Allow timestamp to vary
        calculatedAt: expect.any(Date)
      });
    });
  });
});
```

#### Visual Regression Testing
```javascript
describe('Visual Regression Tests', () => {
  test('should match visual snapshot for login page', async () => {
    const page = await browser.newPage();
    await page.goto('/login');
    
    // Wait for all images and fonts to load
    await page.waitForLoadState('networkidle');
    
    // Take screenshot and compare to golden master
    const screenshot = await page.screenshot({ fullPage: true });
    expect(screenshot).toMatchSnapshot('login-page.png');
  });
  
  test('should match visual snapshot for responsive design', async () => {
    const page = await browser.newPage();
    
    // Test different viewport sizes
    const viewports = [
      { width: 375, height: 667, name: 'mobile' },
      { width: 768, height: 1024, name: 'tablet' },
      { width: 1920, height: 1080, name: 'desktop' }
    ];
    
    for (const viewport of viewports) {
      await page.setViewportSize(viewport);
      await page.goto('/products');
      await page.waitForLoadState('networkidle');
      
      const screenshot = await page.screenshot({ fullPage: true });
      expect(screenshot).toMatchSnapshot(`products-${viewport.name}.png`);
    }
  });
});
```

### 7. BDD with Cucumber/Gherkin

Business-readable tests using natural language specifications.

#### Gherkin Feature Files
```gherkin
# features/user-authentication.feature
Feature: User Authentication
  As a customer
  I want to log into my account
  So that I can access my personal information and order history

  Background:
    Given the application is running
    And the database is clean

  Scenario: Successful login with valid credentials
    Given a user exists with email "customer@example.com" and password "validpassword"
    When the user enters email "customer@example.com"
    And the user enters password "validpassword"
    And the user clicks the login button
    Then the user should be redirected to the dashboard
    And the user should see a welcome message

  Scenario: Failed login with invalid password
    Given a user exists with email "customer@example.com" and password "validpassword"
    When the user enters email "customer@example.com"
    And the user enters password "wrongpassword"
    And the user clicks the login button
    Then the user should see an error message "Invalid credentials"
    And the user should remain on the login page

  Scenario Outline: Login validation for different invalid inputs
    When the user enters email "<email>"
    And the user enters password "<password>"
    And the user clicks the login button
    Then the user should see an error message "<error_message>"

    Examples:
      | email              | password    | error_message        |
      |                    | password123 | Email is required    |
      | invalid-email      | password123 | Invalid email format |
      | user@example.com   |             | Password is required |
```

#### Step Definitions
```javascript
// step-definitions/authentication.steps.js
import { Given, When, Then } from '@cucumber/cucumber';
import { expect } from '@playwright/test';

Given('a user exists with email {string} and password {string}', 
  async function (email, password) {
    await this.testDb.users.create({
      email,
      password: await hashPassword(password),
      isActive: true
    });
  }
);

When('the user enters email {string}', async function (email) {
  await this.page.fill('[data-testid="email-input"]', email);
});

When('the user enters password {string}', async function (password) {
  await this.page.fill('[data-testid="password-input"]', password);
});

When('the user clicks the login button', async function () {
  await this.page.click('[data-testid="login-button"]');
});

Then('the user should be redirected to the dashboard', async function () {
  await this.page.waitForURL('/dashboard');
  expect(this.page.url()).toContain('/dashboard');
});

Then('the user should see a welcome message', async function () {
  const welcomeMessage = await this.page.textContent('[data-testid="welcome-message"]');
  expect(welcomeMessage).toContain('Welcome');
});

Then('the user should see an error message {string}', async function (expectedMessage) {
  const errorMessage = await this.page.textContent('[data-testid="error-message"]');
  expect(errorMessage).toBe(expectedMessage);
});
```

#### World/Context Setup
```javascript
// support/world.js
import { setWorldConstructor, Before, After } from '@cucumber/cucumber';
import { chromium } from '@playwright/test';

class CustomWorld {
  constructor() {
    this.browser = null;
    this.page = null;
    this.testDb = null;
  }
  
  async init() {
    this.browser = await chromium.launch();
    this.page = await this.browser.newPage();
    this.testDb = await createTestDatabase();
  }
  
  async cleanup() {
    if (this.page) await this.page.close();
    if (this.browser) await this.browser.close();
    if (this.testDb) await this.testDb.close();
  }
}

setWorldConstructor(CustomWorld);

Before(async function () {
  await this.init();
});

After(async function () {
  await this.cleanup();
});
```

## 🎯 Choosing the Right Testing Pattern

### Decision Matrix

| Pattern | Best For | Readability | Maintenance | Learning Curve |
|---------|----------|-------------|-------------|----------------|
| **AAA** | Unit tests, technical teams | High | Low | Low |
| **GWT** | BDD, business stakeholders | Very High | Medium | Medium |
| **SEAT** | Integration/E2E, complex setup | Medium | High | Medium |
| **Four-Phase** | Complex test suites | High | Medium | Low |
| **Property-Based** | Edge cases, mathematical logic | Medium | Medium | High |
| **Golden Master** | Legacy code, complex outputs | Low | High | Low |
| **BDD/Gherkin** | Business requirements | Very High | High | High |

### Pattern Selection Guide

#### For Unit Tests
- **Simple logic:** AAA pattern
- **Business rules:** GWT pattern
- **Mathematical functions:** Property-based testing
- **Complex setup:** Four-phase pattern

#### For Integration Tests
- **API testing:** SEAT pattern
- **Component integration:** AAA or GWT
- **Database operations:** SEAT pattern
- **Legacy systems:** Golden Master

#### For E2E Tests
- **User journeys:** GWT or BDD
- **Critical paths:** SEAT pattern
- **Visual testing:** Golden Master
- **Cross-browser:** Property-based with browsers

## 💡 Pattern Combination Strategies

### Hybrid Approach: AAA + Property-Based
```javascript
describe('User Input Validation', () => {
  // AAA for specific examples
  test('should validate common email formats', () => {
    // Arrange
    const validEmails = [
      'user@example.com',
      'user.name@example.co.uk',
      'user+tag@example.org'
    ];
    
    // Act & Assert
    validEmails.forEach(email => {
      expect(isValidEmail(email)).toBe(true);
    });
  });
  
  // Property-based for edge cases
  test('should handle random email-like strings', () => {
    fc.assert(
      fc.property(
        fc.emailAddress(),
        (email) => {
          // Property: Generated emails should always validate
          expect(isValidEmail(email)).toBe(true);
        }
      )
    );
  });
});
```

### Layered Testing: AAA + GWT + SEAT
```javascript
// Unit level (AAA)
describe('CartCalculator (Unit)', () => {
  test('should calculate tax correctly', () => {
    // Arrange
    const calculator = new CartCalculator();
    const item = { price: 100, taxable: true };
    const taxRate = 0.08;
    
    // Act
    const tax = calculator.calculateTax(item, taxRate);
    
    // Assert
    expect(tax).toBe(8);
  });
});

// Integration level (GWT)
describe('Shopping Cart (Integration)', () => {
  test('Given items in cart, When user applies discount, Then total should reflect discount', () => {
    // Given
    const cart = new ShoppingCart();
    cart.addItem({ id: 1, price: 100, quantity: 2 });
    
    // When
    cart.applyDiscount({ type: 'percentage', value: 10 });
    
    // Then
    expect(cart.getTotal()).toBe(180); // 200 - 20 discount
  });
});

// E2E level (SEAT)
describe('Checkout Flow (E2E)', () => {
  let browser, page, testData;
  
  beforeAll(async () => {
    // Setup
    browser = await chromium.launch();
    testData = await createTestData();
  });
  
  afterAll(async () => {
    // Teardown
    await browser.close();
    await cleanupTestData(testData);
  });
  
  test('should complete full checkout process', async () => {
    // Setup
    page = await browser.newPage();
    await setupCartWithItems(page, testData.products);
    
    // Execute
    await completeCheckoutFlow(page, testData.user);
    
    // Assert
    await verifyOrderCreated(page, testData.user);
    
    // Teardown handled by afterAll
  });
});
```

## 📝 Exercise: Pattern Application

Choose a feature from your current project and implement tests using different patterns:

### Step 1: Feature Analysis
Pick a feature that involves:
- User input
- Business logic
- API calls
- UI updates

### Step 2: Pattern Mapping
- **Unit tests:** Use AAA for pure functions
- **Integration tests:** Use GWT for user workflows
- **E2E tests:** Use SEAT for complete flows
- **Edge cases:** Add property-based testing

### Step 3: Implementation
Write test suites demonstrating each pattern.

### Step 4: Comparison
Document what you learned about each pattern's strengths and weaknesses.

**Deliverable:** A test suite showcasing at least 3 different patterns applied to the same feature.

---

**Next:** [Module 4: Frontend Testing in Practice](../04-testing-in-practice/) - Apply these patterns using modern testing tools and frameworks.