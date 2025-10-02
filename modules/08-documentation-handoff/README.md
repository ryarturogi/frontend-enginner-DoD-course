# Module 8: Documentation & Handoff
*Master comprehensive documentation and seamless team handoffs with modern practices*

## 🎯 Learning Objectives

By the end of this module, you will:
- **Create comprehensive technical documentation** that reduces maintenance burden and accelerates onboarding
- **Write effective Architecture Decision Records (ADRs)** for future reference and decision tracking
- **Implement self-documenting code practices** with TypeScript, JSDoc, and automated documentation generation
- **Design effective handoff processes** for QA, DevOps, and cross-functional team members
- **Build maintainable codebases** that new team members can quickly understand and contribute to
- **Create living documentation** that evolves automatically with your codebase using modern tooling
- **Implement documentation-driven development** with API-first and contract-first approaches
- **Set up automated documentation workflows** with continuous integration and deployment

## 📚 Technical Documentation Best Practices

### 1. README Excellence

A great README is the first line of defense against confusion and the gateway to your project.

```markdown
# Project Name - Production-Ready E-commerce Platform

## Overview

A modern, scalable e-commerce platform built with React, TypeScript, and Next.js. This application handles high-traffic scenarios with optimized performance, comprehensive testing, and enterprise-grade security.

## Quick Start

```bash
# Prerequisites: Node.js 18+, pnpm 8+
git clone <repository-url>
cd project-name
cp .env.example .env.local
pnpm install
pnpm dev
```

Visit [http://localhost:3000](http://localhost:3000) to see the application.

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend API   │    │   Database      │
│   (Next.js)     │◄──►│   (Node.js)     │◄──►│   (PostgreSQL)  │
│   Port: 3000    │    │   Port: 8000    │    │   Port: 5432    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CDN           │    │   Redis Cache   │    │   File Storage  │
│   (CloudFlare)  │    │   Port: 6379    │    │   (AWS S3)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## Technology Stack

### Core Technologies
- **Frontend**: React 19, TypeScript 5, Next.js 15
- **Styling**: Tailwind CSS 4, CSS Modules
- **State Management**: Zustand, React Query
- **Testing**: Jest, React Testing Library, Playwright
- **Build Tools**: Turbo, ESBuild, SWC

### Development Tools
- **Code Quality**: ESLint, Prettier, Husky
- **Type Checking**: TypeScript strict mode
- **Performance**: Lighthouse CI, Bundle Analyzer
- **Monitoring**: Sentry, PostHog

## Project Structure

```
src/
├── components/          # Reusable UI components
│   ├── ui/             # Basic UI primitives
│   ├── forms/          # Form components
│   └── layout/         # Layout components
├── pages/              # Next.js pages
├── hooks/              # Custom React hooks
├── lib/                # Utility functions
├── services/           # API services
├── stores/             # State management
├── styles/             # Global styles
├── types/              # TypeScript definitions
└── utils/              # Helper functions
```

## Environment Setup

### Required Environment Variables

```bash
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# Authentication
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# External Services
STRIPE_SECRET_KEY="sk_test_..."
SENDGRID_API_KEY="SG...."

# Monitoring
SENTRY_DSN="https://...@sentry.io/..."
```

### Development Environment

1. **Database Setup**
   ```bash
   # Using Docker
   docker-compose up -d postgres redis
   
   # Run migrations
   pnpm db:migrate
   pnpm db:seed
   ```

2. **Testing Setup**
   ```bash
   # Install Playwright browsers
   npx playwright install
   
   # Run tests
   pnpm test
   pnpm test:e2e
   ```

## Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start development server |
| `pnpm build` | Build for production |
| `pnpm start` | Start production server |
| `pnpm test` | Run unit tests |
| `pnpm test:e2e` | Run end-to-end tests |
| `pnpm test:coverage` | Run tests with coverage |
| `pnpm lint` | Lint code |
| `pnpm type-check` | Check TypeScript types |
| `pnpm db:migrate` | Run database migrations |
| `pnpm db:seed` | Seed database with test data |

## API Documentation

### Authentication

All API endpoints require authentication except for public product data.

```typescript
// Include JWT token in Authorization header
headers: {
  'Authorization': 'Bearer <jwt-token>',
  'Content-Type': 'application/json'
}
```

### Core Endpoints

#### Products
- `GET /api/products` - List products with filtering
- `GET /api/products/[id]` - Get product details
- `POST /api/products` - Create product (admin only)

#### Users
- `GET /api/users/profile` - Get user profile
- `PUT /api/users/profile` - Update user profile
- `POST /api/users/register` - Register new user

See [API Documentation](./docs/api.md) for complete endpoint reference.

## Performance Standards

- **Lighthouse Score**: >90 for all metrics
- **Core Web Vitals**: LCP <2.5s, FID <100ms, CLS <0.1
- **Bundle Size**: <500KB initial bundle
- **Test Coverage**: >80% for critical paths

## Security Considerations

- Input validation on all user data
- CSRF protection on state-changing operations
- Content Security Policy (CSP) configured
- Secrets managed via environment variables
- Regular dependency security audits

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes following our [coding standards](./docs/coding-standards.md)
4. Add tests for new functionality
5. Ensure all tests pass (`pnpm test`)
6. Update documentation as needed
7. Commit changes (`git commit -m 'Add amazing feature'`)
8. Push to branch (`git push origin feature/amazing-feature`)
9. Open a Pull Request

## Deployment

### Staging Environment
- **URL**: https://staging.myapp.com
- **Deploy**: Automatic on merge to `develop` branch
- **Database**: Staging PostgreSQL instance

### Production Environment
- **URL**: https://myapp.com
- **Deploy**: Manual approval required
- **Database**: Production PostgreSQL with replicas

See [Deployment Guide](./docs/deployment.md) for detailed instructions.

## Troubleshooting

### Common Issues

**Build Errors**
```bash
# Clear cache and reinstall
rm -rf .next node_modules
pnpm install
pnpm build
```

**Database Connection Issues**
```bash
# Check database status
docker-compose ps
# Reset database
pnpm db:reset
```

**Test Failures**
```bash
# Update snapshots
pnpm test -- --updateSnapshot
# Run specific test
pnpm test -- --testNamePattern="Login"
```

## Support

- **Documentation**: [Project Wiki](./docs/)
- **Issues**: [GitHub Issues](https://github.com/org/project/issues)
- **Slack**: #project-name channel
- **Team Lead**: @john.doe

## License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.
```

### 2. Component Documentation

```typescript
/**
 * ProductCard - Displays product information in a card format
 * 
 * @example
 * ```tsx
 * <ProductCard
 *   product={{
 *     id: "1",
 *     name: "Wireless Headphones",
 *     price: 199.99,
 *     image: "/images/headphones.jpg",
 *     rating: 4.5
 *   }}
 *   onAddToCart={(product) => console.log('Added to cart:', product)}
 *   variant="default"
 * />
 * ```
 */

interface ProductCardProps {
  /** Product data to display */
  product: {
    /** Unique product identifier */
    id: string;
    /** Product name (max 100 characters) */
    name: string;
    /** Product price in USD */
    price: number;
    /** Product image URL */
    image: string;
    /** Average rating (0-5) */
    rating: number;
    /** Whether product is in stock */
    inStock?: boolean;
    /** Original price for sale items */
    originalPrice?: number;
  };
  
  /** Called when user clicks "Add to Cart" button */
  onAddToCart: (product: ProductCardProps['product']) => void;
  
  /** Visual variant of the card */
  variant?: 'default' | 'compact' | 'featured';
  
  /** Whether to show the "Add to Cart" button */
  showAddToCart?: boolean;
  
  /** Additional CSS class names */
  className?: string;
}

/**
 * ProductCard component renders a product in card format with image,
 * name, price, rating, and optional add-to-cart functionality.
 * 
 * Features:
 * - Responsive design that works on mobile and desktop
 * - Accessibility support with proper ARIA labels
 * - Loading states for images
 * - Sale price display when applicable
 * - Stock status indicators
 * 
 * Performance:
 * - Images are lazy loaded
 * - Optimized for rendering in lists
 * - Minimal re-renders with React.memo
 * 
 * @param props - Component props
 * @returns Rendered product card
 */
export const ProductCard: React.FC<ProductCardProps> = React.memo(({
  product,
  onAddToCart,
  variant = 'default',
  showAddToCart = true,
  className
}) => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [imageError, setImageError] = useState(false);
  
  const isOnSale = product.originalPrice && product.originalPrice > product.price;
  const discountPercentage = isOnSale 
    ? Math.round(((product.originalPrice - product.price) / product.originalPrice) * 100)
    : 0;
  
  const handleAddToCart = useCallback(() => {
    if (product.inStock !== false) {
      onAddToCart(product);
      
      // Track analytics event
      trackEvent('add_to_cart', {
        product_id: product.id,
        product_name: product.name,
        price: product.price
      });
    }
  }, [product, onAddToCart]);
  
  const cardClassName = clsx(
    'product-card',
    `product-card--${variant}`,
    {
      'product-card--out-of-stock': product.inStock === false,
      'product-card--on-sale': isOnSale
    },
    className
  );
  
  return (
    <article className={cardClassName} data-testid="product-card">
      <div className="product-card__image-container">
        {!imageLoaded && !imageError && (
          <div className="product-card__image-skeleton" aria-hidden="true" />
        )}
        
        {!imageError ? (
          <img
            src={product.image}
            alt={product.name}
            className="product-card__image"
            loading="lazy"
            onLoad={() => setImageLoaded(true)}
            onError={() => setImageError(true)}
          />
        ) : (
          <div className="product-card__image-fallback" aria-label="Image not available">
            <ImageIcon />
          </div>
        )}
        
        {isOnSale && (
          <div className="product-card__sale-badge" aria-label={`${discountPercentage}% off`}>
            -{discountPercentage}%
          </div>
        )}
        
        {product.inStock === false && (
          <div className="product-card__stock-overlay">
            <span>Out of Stock</span>
          </div>
        )}
      </div>
      
      <div className="product-card__content">
        <h3 className="product-card__name">{product.name}</h3>
        
        <div className="product-card__rating" aria-label={`Rating: ${product.rating} out of 5 stars`}>
          <StarRating rating={product.rating} readonly />
          <span className="sr-only">{product.rating} out of 5 stars</span>
        </div>
        
        <div className="product-card__price">
          <span className="product-card__current-price" aria-label={`Current price: $${product.price}`}>
            ${product.price.toFixed(2)}
          </span>
          
          {isOnSale && (
            <span 
              className="product-card__original-price"
              aria-label={`Original price: $${product.originalPrice}`}
            >
              ${product.originalPrice!.toFixed(2)}
            </span>
          )}
        </div>
        
        {showAddToCart && (
          <button
            type="button"
            className="product-card__add-to-cart"
            onClick={handleAddToCart}
            disabled={product.inStock === false}
            aria-label={`Add ${product.name} to cart`}
          >
            {product.inStock === false ? 'Out of Stock' : 'Add to Cart'}
          </button>
        )}
      </div>
    </article>
  );
});

ProductCard.displayName = 'ProductCard';
```

### 3. API Documentation

```typescript
/**
 * @fileoverview User API service
 * 
 * This module provides functions for interacting with user-related endpoints.
 * All functions handle authentication, error handling, and response validation.
 * 
 * @author Engineering Team
 * @since 1.0.0
 */

import { z } from 'zod';
import { ApiClient } from './api-client';

// Request/Response schemas for validation and documentation

/**
 * User registration request schema
 */
export const RegisterUserRequest = z.object({
  /** User's email address (must be unique) */
  email: z.string().email('Invalid email format').max(254),
  
  /** User's password (minimum 8 characters, must contain uppercase, lowercase, number, and symbol) */
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*])/, 'Password must contain uppercase, lowercase, number, and special character'),
  
  /** User's full name */
  name: z.string().min(2, 'Name must be at least 2 characters').max(100),
  
  /** Optional phone number */
  phone: z.string().optional(),
  
  /** Whether user agrees to terms of service */
  acceptTerms: z.boolean().refine(val => val === true, 'Must accept terms of service')
});

/**
 * User profile response schema
 */
export const UserProfile = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string(),
  phone: z.string().nullable(),
  avatar: z.string().url().nullable(),
  emailVerified: z.boolean(),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime(),
  preferences: z.object({
    newsletter: z.boolean(),
    notifications: z.boolean(),
    theme: z.enum(['light', 'dark', 'auto'])
  })
});

/**
 * User profile update request schema
 */
export const UpdateUserProfileRequest = z.object({
  name: z.string().min(2).max(100).optional(),
  phone: z.string().optional(),
  preferences: z.object({
    newsletter: z.boolean().optional(),
    notifications: z.boolean().optional(),
    theme: z.enum(['light', 'dark', 'auto']).optional()
  }).optional()
});

// Type definitions
export type RegisterUserRequestType = z.infer<typeof RegisterUserRequest>;
export type UserProfileType = z.infer<typeof UserProfile>;
export type UpdateUserProfileRequestType = z.infer<typeof UpdateUserProfileRequest>;

/**
 * User API service class
 * 
 * Provides methods for user management including registration, profile management,
 * and user preferences. All methods include proper error handling and request validation.
 * 
 * @example
 * ```typescript
 * const userApi = new UserApi();
 * 
 * // Register new user
 * const user = await userApi.register({
 *   email: 'user@example.com',
 *   password: 'SecurePass123!',
 *   name: 'John Doe',
 *   acceptTerms: true
 * });
 * 
 * // Get user profile
 * const profile = await userApi.getProfile();
 * 
 * // Update user preferences
 * await userApi.updateProfile({
 *   preferences: { theme: 'dark' }
 * });
 * ```
 */
export class UserApi {
  private apiClient: ApiClient;
  
  constructor() {
    this.apiClient = new ApiClient('/api/users');
  }
  
  /**
   * Register a new user account
   * 
   * Creates a new user account with the provided information. Sends a verification
   * email if email verification is enabled. Returns the created user profile.
   * 
   * @param userData - User registration data
   * @returns Promise resolving to created user profile
   * 
   * @throws {ValidationError} When request data is invalid
   * @throws {ConflictError} When email already exists
   * @throws {ApiError} When registration fails for other reasons
   * 
   * @example
   * ```typescript
   * try {
   *   const user = await userApi.register({
   *     email: 'new-user@example.com',
   *     password: 'SecurePassword123!',
   *     name: 'Jane Smith',
   *     acceptTerms: true
   *   });
   *   console.log('User registered:', user.id);
   * } catch (error) {
   *   if (error instanceof ConflictError) {
   *     console.error('Email already registered');
   *   } else {
   *     console.error('Registration failed:', error.message);
   *   }
   * }
   * ```
   */
  async register(userData: RegisterUserRequestType): Promise<UserProfileType> {
    // Validate request data
    const validatedData = RegisterUserRequest.parse(userData);
    
    const response = await this.apiClient.post('/register', validatedData);
    
    // Validate response data
    return UserProfile.parse(response.data);
  }
  
  /**
   * Get current user's profile
   * 
   * Retrieves the authenticated user's complete profile information including
   * preferences and account status.
   * 
   * @returns Promise resolving to user profile
   * 
   * @throws {UnauthorizedError} When user is not authenticated
   * @throws {NotFoundError} When user profile is not found
   * @throws {ApiError} When request fails
   * 
   * @example
   * ```typescript
   * try {
   *   const profile = await userApi.getProfile();
   *   console.log('Current user:', profile.name);
   * } catch (error) {
   *   if (error instanceof UnauthorizedError) {
   *     // Redirect to login
   *     window.location.href = '/login';
   *   }
   * }
   * ```
   */
  async getProfile(): Promise<UserProfileType> {
    const response = await this.apiClient.get('/profile');
    return UserProfile.parse(response.data);
  }
  
  /**
   * Update user profile
   * 
   * Updates the authenticated user's profile with the provided data. Only
   * specified fields will be updated; omitted fields remain unchanged.
   * 
   * @param updates - Partial user profile data to update
   * @returns Promise resolving to updated user profile
   * 
   * @throws {ValidationError} When update data is invalid
   * @throws {UnauthorizedError} When user is not authenticated
   * @throws {ApiError} When update fails
   * 
   * @example
   * ```typescript
   * // Update user name
   * await userApi.updateProfile({ name: 'New Name' });
   * 
   * // Update preferences
   * await userApi.updateProfile({
   *   preferences: {
   *     theme: 'dark',
   *     newsletter: false
   *   }
   * });
   * ```
   */
  async updateProfile(updates: UpdateUserProfileRequestType): Promise<UserProfileType> {
    // Validate update data
    const validatedUpdates = UpdateUserProfileRequest.parse(updates);
    
    const response = await this.apiClient.put('/profile', validatedUpdates);
    
    // Validate response data
    return UserProfile.parse(response.data);
  }
  
  /**
   * Upload user avatar
   * 
   * Uploads a new avatar image for the authenticated user. Image will be
   * resized and optimized automatically.
   * 
   * @param file - Image file to upload (JPEG, PNG, WebP)
   * @param onProgress - Optional progress callback
   * @returns Promise resolving to updated user profile with new avatar URL
   * 
   * @throws {ValidationError} When file is invalid (size, type, etc.)
   * @throws {UnauthorizedError} When user is not authenticated
   * @throws {ApiError} When upload fails
   * 
   * @example
   * ```typescript
   * const handleAvatarUpload = async (file: File) => {
   *   try {
   *     const updatedProfile = await userApi.uploadAvatar(
   *       file,
   *       (progress) => console.log(`Upload: ${progress}%`)
   *     );
   *     console.log('New avatar URL:', updatedProfile.avatar);
   *   } catch (error) {
   *     console.error('Avatar upload failed:', error.message);
   *   }
   * };
   * ```
   */
  async uploadAvatar(
    file: File, 
    onProgress?: (progress: number) => void
  ): Promise<UserProfileType> {
    // Validate file
    if (!['image/jpeg', 'image/png', 'image/webp'].includes(file.type)) {
      throw new ValidationError('Invalid file type. Only JPEG, PNG, and WebP are allowed.');
    }
    
    if (file.size > 5 * 1024 * 1024) { // 5MB limit
      throw new ValidationError('File size must be less than 5MB.');
    }
    
    const formData = new FormData();
    formData.append('avatar', file);
    
    const response = await this.apiClient.post('/avatar', formData, {
      onUploadProgress: (event) => {
        if (onProgress && event.total) {
          const progress = Math.round((event.loaded * 100) / event.total);
          onProgress(progress);
        }
      }
    });
    
    return UserProfile.parse(response.data);
  }
  
  /**
   * Delete user account
   * 
   * Permanently deletes the authenticated user's account and all associated data.
   * This action cannot be undone.
   * 
   * @param confirmation - Must be the string "DELETE" to confirm deletion
   * @returns Promise resolving when account is deleted
   * 
   * @throws {ValidationError} When confirmation is incorrect
   * @throws {UnauthorizedError} When user is not authenticated
   * @throws {ApiError} When deletion fails
   * 
   * @example
   * ```typescript
   * try {
   *   await userApi.deleteAccount('DELETE');
   *   console.log('Account deleted successfully');
   *   // Redirect to homepage
   *   window.location.href = '/';
   * } catch (error) {
   *   console.error('Account deletion failed:', error.message);
   * }
   * ```
   */
  async deleteAccount(confirmation: string): Promise<void> {
    if (confirmation !== 'DELETE') {
      throw new ValidationError('Must confirm deletion by typing "DELETE"');
    }
    
    await this.apiClient.delete('/profile', { confirmation });
  }
}

// Export singleton instance
export const userApi = new UserApi();
```

## 🏗️ Architecture Decision Records (ADRs)

ADRs document important architectural decisions for future reference and onboarding.

### ADR Template

```markdown
# ADR-001: Choose State Management Solution

## Status
Accepted

## Context
We need to choose a state management solution for our React application that will:
- Handle complex application state across multiple components
- Provide good developer experience and debugging tools
- Scale with our team and application growth
- Integrate well with our existing React and TypeScript setup
- Support server-side rendering with Next.js

## Decision
We will use Zustand for client-side state management, complemented by React Query for server state.

## Rationale

### Considered Options
1. **Redux Toolkit** - Industry standard, extensive ecosystem
2. **Zustand** - Lightweight, TypeScript-first, minimal boilerplate
3. **React Context** - Built-in, simple for basic use cases
4. **Valtio** - Proxy-based, immutable by default

### Zustand Advantages
- **Minimal boilerplate**: No providers, reducers, or action creators required
- **TypeScript support**: Excellent TypeScript integration out of the box
- **Small bundle size**: ~13KB compared to Redux's ~47KB
- **Flexible**: Works with or without React, supports middleware
- **Simple testing**: Easy to test stores in isolation
- **DevTools**: Good integration with Redux DevTools

### React Query for Server State
- **Caching**: Automatic caching with intelligent cache invalidation
- **Background refetching**: Keeps data fresh automatically
- **Error handling**: Built-in error and loading states
- **Optimistic updates**: Easy to implement optimistic UI patterns
- **SSR support**: Works seamlessly with Next.js

### Implementation Example

```typescript
// User store with Zustand
interface UserState {
  user: User | null;
  setUser: (user: User | null) => void;
  updateProfile: (updates: Partial<User>) => void;
}

export const useUserStore = create<UserState>((set, get) => ({
  user: null,
  setUser: (user) => set({ user }),
  updateProfile: (updates) => set((state) => ({
    user: state.user ? { ...state.user, ...updates } : null
  }))
}));

// Server state with React Query
export const useProducts = (filters: ProductFilters) => {
  return useQuery(['products', filters], () => fetchProducts(filters), {
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
};
```

## Consequences

### Positive
- Reduced boilerplate code compared to Redux
- Better developer experience with TypeScript
- Smaller bundle size improves performance
- Clear separation between client and server state
- Easier onboarding for new developers
- Built-in performance optimizations

### Negative
- Smaller ecosystem compared to Redux
- Less mature (though stable) compared to Redux
- Team needs to learn new patterns
- May need custom solutions for some advanced use cases

### Mitigation Strategies
- Create documentation and examples for common patterns
- Establish conventions for store organization
- Provide training sessions for team members
- Monitor performance and developer experience

## Compliance
This decision aligns with our architectural principles:
- **Simplicity**: Zustand reduces complexity
- **Performance**: Smaller bundle size and optimizations
- **Developer Experience**: Better TypeScript support and less boilerplate
- **Maintainability**: Clear separation of concerns

## Related Decisions
- ADR-002: API Layer Architecture (React Query usage)
- ADR-003: Component Architecture (state lifting patterns)

## References
- [Zustand Documentation](https://zustand-demo.pmnd.rs/)
- [React Query Documentation](https://tanstack.com/query/latest)
- [State Management Comparison](https://github.com/pmndrs/zustand#comparison)

---

**Date**: 2024-01-15
**Author**: Senior Engineering Team
**Reviewers**: Tech Lead, Principal Engineer
**Next Review**: 2024-07-15 (6 months)
```

### Real-World ADRs

```markdown
# ADR-002: API Client Architecture

## Status
Accepted

## Context
We need a consistent approach to making API calls throughout our application that provides:
- Type safety for all API interactions
- Consistent error handling and retry logic
- Request/response logging and monitoring
- Authentication token management
- Request deduplication and caching
- Support for file uploads and downloads

## Decision
Implement a layered API architecture:
1. **Base API Client** - HTTP client with common functionality
2. **Service Layer** - Typed API services for each domain
3. **React Query Integration** - Caching and synchronization
4. **Custom Hooks** - Component-level API integration

## Implementation

### 1. Base API Client
```typescript
export class ApiClient {
  private baseUrl: string;
  private defaultHeaders: Record<string, string>;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      'X-Requested-With': 'XMLHttpRequest'
    };
  }
  
  async request<T>(endpoint: string, options: RequestOptions = {}): Promise<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;
    const config = this.buildRequestConfig(options);
    
    try {
      const response = await fetch(url, config);
      return this.handleResponse<T>(response);
    } catch (error) {
      throw this.handleError(error);
    }
  }
  
  private async handleResponse<T>(response: Response): Promise<ApiResponse<T>> {
    if (!response.ok) {
      const error = await this.parseErrorResponse(response);
      throw new ApiError(error.message, response.status, error);
    }
    
    const data = await response.json();
    return { data, status: response.status, headers: response.headers };
  }
}
```

### 2. Service Layer
```typescript
export class ProductService {
  constructor(private apiClient: ApiClient) {}
  
  async getProducts(filters: ProductFilters): Promise<ProductList> {
    const params = new URLSearchParams(filters);
    const response = await this.apiClient.get<ProductList>(`/products?${params}`);
    return ProductListSchema.parse(response.data);
  }
  
  async getProduct(id: string): Promise<Product> {
    const response = await this.apiClient.get<Product>(`/products/${id}`);
    return ProductSchema.parse(response.data);
  }
}
```

### 3. React Query Integration
```typescript
export const useProducts = (filters: ProductFilters) => {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => productService.getProducts(filters),
    staleTime: 5 * 60 * 1000,
    select: (data) => data.products,
    onError: (error) => {
      console.error('Failed to fetch products:', error);
      toast.error('Failed to load products');
    }
  });
};
```

## Consequences

### Positive
- Type safety across all API interactions
- Consistent error handling and user feedback
- Automatic request deduplication and caching
- Easy testing with service layer isolation
- Built-in loading and error states

### Negative
- Additional abstraction layers increase complexity
- Learning curve for team members
- Potential over-engineering for simple requests

## Metrics
- API error rate < 1%
- Average API response time < 500ms
- Developer satisfaction with API integration > 8/10

---

# ADR-003: Component Testing Strategy

## Status
Accepted

## Context
We need a comprehensive testing strategy for React components that ensures:
- Reliable user-facing functionality
- Maintainable test suite that doesn't break with refactoring
- Good performance (fast test execution)
- High confidence in production deployments

## Decision
Implement a three-layer testing strategy:
1. **Unit Tests** (70%) - Individual component behavior
2. **Integration Tests** (20%) - Component interactions
3. **E2E Tests** (10%) - Complete user workflows

## Testing Principles

### Test User Behavior, Not Implementation
```typescript
// ❌ Testing implementation details
expect(component.state.isLoading).toBe(true);
expect(mockSetState).toHaveBeenCalledWith({ isLoading: true });

// ✅ Testing user-visible behavior
expect(screen.getByText('Loading...')).toBeInTheDocument();
expect(screen.getByRole('button')).toBeDisabled();
```

### Use Semantic Queries
```typescript
// ❌ Fragile selectors
screen.getByTestId('submit-btn');
container.querySelector('.form-submit-button');

// ✅ Semantic queries
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText('Email address');
```

### Test Error Boundaries
```typescript
describe('ProductCard Error Handling', () => {
  it('should handle image loading errors gracefully', async () => {
    // Simulate image error
    mockImageLoad.mockRejectedValue(new Error('Image not found'));
    
    render(<ProductCard product={mockProduct} />);
    
    // Should show fallback image
    expect(await screen.findByAltText('Image not available')).toBeInTheDocument();
    expect(screen.queryByRole('img')).not.toBeInTheDocument();
  });
});
```

## Implementation Guidelines

### Component Test Structure
```typescript
describe('UserProfileForm', () => {
  // Test data setup
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com'
  };
  
  const defaultProps = {
    user: mockUser,
    onSubmit: jest.fn(),
    onCancel: jest.fn()
  };
  
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  describe('Rendering', () => {
    it('should display user information correctly', () => {
      render(<UserProfileForm {...defaultProps} />);
      
      expect(screen.getByDisplayValue('John Doe')).toBeInTheDocument();
      expect(screen.getByDisplayValue('john@example.com')).toBeInTheDocument();
    });
  });
  
  describe('User Interactions', () => {
    it('should submit form with updated data', async () => {
      const user = userEvent.setup();
      render(<UserProfileForm {...defaultProps} />);
      
      // User updates name
      const nameInput = screen.getByLabelText('Full Name');
      await user.clear(nameInput);
      await user.type(nameInput, 'Jane Doe');
      
      // User submits form
      await user.click(screen.getByRole('button', { name: /save/i }));
      
      // Verify submission
      expect(defaultProps.onSubmit).toHaveBeenCalledWith({
        ...mockUser,
        name: 'Jane Doe'
      });
    });
  });
  
  describe('Error Handling', () => {
    it('should display validation errors', async () => {
      const user = userEvent.setup();
      render(<UserProfileForm {...defaultProps} />);
      
      // Clear required field
      const nameInput = screen.getByLabelText('Full Name');
      await user.clear(nameInput);
      await user.click(screen.getByRole('button', { name: /save/i }));
      
      // Should show error
      expect(await screen.findByText('Name is required')).toBeInTheDocument();
      expect(defaultProps.onSubmit).not.toHaveBeenCalled();
    });
  });
});
```

## Consequences

### Positive
- High confidence in component behavior
- Tests survive refactoring (testing behavior, not implementation)
- Fast feedback during development
- Good documentation of component capabilities

### Negative
- Higher initial investment in test setup
- Learning curve for testing-library patterns
- Some integration tests can be slower

## Success Metrics
- Test coverage > 80% for critical components
- Test execution time < 30 seconds for unit tests
- E2E test suite < 10 minutes
- Low false positive rate (< 5% flaky tests)

---

Date: 2024-01-20
Author: QA Team Lead
Next Review: 2024-07-20
```

## 📝 Self-Documenting Code with TypeScript

### 1. Comprehensive Type Definitions

```typescript
/**
 * @fileoverview Product domain types and utilities
 * 
 * This file contains all type definitions related to products,
 * including schemas, validation, and utility types.
 */

/**
 * Product availability status
 */
export enum ProductStatus {
  /** Product is available for purchase */
  AVAILABLE = 'available',
  /** Product is out of stock */
  OUT_OF_STOCK = 'out_of_stock',
  /** Product is discontinued */
  DISCONTINUED = 'discontinued',
  /** Product is coming soon */
  COMING_SOON = 'coming_soon'
}

/**
 * Product category classification
 */
export enum ProductCategory {
  ELECTRONICS = 'electronics',
  CLOTHING = 'clothing',
  BOOKS = 'books',
  HOME_GARDEN = 'home_garden',
  SPORTS = 'sports',
  TOYS = 'toys'
}

/**
 * Base product information that all products must have
 */
interface BaseProduct {
  /** Unique product identifier (UUID v4) */
  readonly id: string;
  
  /** Product name (1-200 characters) */
  name: string;
  
  /** Product description in markdown format */
  description: string;
  
  /** Product category */
  category: ProductCategory;
  
  /** Current availability status */
  status: ProductStatus;
  
  /** When the product was first created */
  readonly createdAt: Date;
  
  /** When the product was last updated */
  readonly updatedAt: Date;
}

/**
 * Pricing information for a product
 */
interface ProductPricing {
  /** Current price in cents (to avoid floating point errors) */
  priceInCents: number;
  
  /** Currency code (ISO 4217) */
  currency: string;
  
  /** Original price before any discounts (optional) */
  originalPriceInCents?: number;
  
  /** Tax rate as decimal (e.g., 0.08 for 8%) */
  taxRate: number;
}

/**
 * Product media assets
 */
interface ProductMedia {
  /** Primary product image URL */
  primaryImage: string;
  
  /** Additional product images */
  images: string[];
  
  /** Product videos (if any) */
  videos?: string[];
  
  /** 360-degree view images */
  threeDImages?: string[];
}

/**
 * Inventory tracking information
 */
interface ProductInventory {
  /** Current stock quantity */
  quantity: number;
  
  /** Minimum stock level before reordering */
  lowStockThreshold: number;
  
  /** SKU (Stock Keeping Unit) identifier */
  sku: string;
  
  /** Barcode/UPC if available */
  barcode?: string;
  
  /** Warehouse location */
  location?: string;
}

/**
 * Product SEO and marketing information
 */
interface ProductSEO {
  /** SEO-friendly URL slug */
  slug: string;
  
  /** Meta title for SEO */
  metaTitle: string;
  
  /** Meta description for SEO */
  metaDescription: string;
  
  /** Keywords for search */
  keywords: string[];
  
  /** Featured product flag */
  featured: boolean;
}

/**
 * Product ratings and reviews summary
 */
interface ProductRatings {
  /** Average rating (0-5) */
  averageRating: number;
  
  /** Total number of reviews */
  reviewCount: number;
  
  /** Rating distribution [1-star, 2-star, 3-star, 4-star, 5-star] */
  ratingDistribution: [number, number, number, number, number];
}

/**
 * Complete product entity with all information
 * 
 * This is the main product type used throughout the application.
 * It combines all product-related information into a single interface.
 * 
 * @example
 * ```typescript
 * const product: Product = {
 *   id: '123e4567-e89b-12d3-a456-426614174000',
 *   name: 'Wireless Bluetooth Headphones',
 *   description: '## Premium Sound Quality\n\nExperience crystal-clear audio...',
 *   category: ProductCategory.ELECTRONICS,
 *   status: ProductStatus.AVAILABLE,
 *   pricing: {
 *     priceInCents: 19999, // $199.99
 *     currency: 'USD',
 *     taxRate: 0.08
 *   },
 *   media: {
 *     primaryImage: '/images/headphones-main.jpg',
 *     images: ['/images/headphones-1.jpg', '/images/headphones-2.jpg']
 *   },
 *   inventory: {
 *     quantity: 50,
 *     lowStockThreshold: 10,
 *     sku: 'WBH-001'
 *   },
 *   seo: {
 *     slug: 'wireless-bluetooth-headphones',
 *     metaTitle: 'Premium Wireless Bluetooth Headphones | AudioStore',
 *     metaDescription: 'High-quality wireless headphones with noise cancellation',
 *     keywords: ['headphones', 'wireless', 'bluetooth', 'audio'],
 *     featured: true
 *   },
 *   ratings: {
 *     averageRating: 4.5,
 *     reviewCount: 128,
 *     ratingDistribution: [2, 5, 15, 45, 61]
 *   },
 *   createdAt: new Date('2024-01-01'),
 *   updatedAt: new Date('2024-01-15')
 * };
 * ```
 */
export interface Product extends BaseProduct {
  pricing: ProductPricing;
  media: ProductMedia;
  inventory: ProductInventory;
  seo: ProductSEO;
  ratings: ProductRatings;
}

/**
 * Simplified product data for list displays
 * 
 * Contains only essential information needed for product cards
 * and list views to improve performance.
 */
export type ProductSummary = Pick<Product, 
  | 'id' 
  | 'name' 
  | 'status' 
  | 'category'
> & {
  /** Current price in cents */
  priceInCents: number;
  /** Primary image URL */
  primaryImage: string;
  /** Average rating */
  averageRating: number;
  /** Whether product is on sale */
  onSale: boolean;
};

/**
 * Product data for creation (excludes readonly fields)
 */
export type CreateProductRequest = Omit<Product, 
  | 'id' 
  | 'createdAt' 
  | 'updatedAt'
  | 'ratings'
>;

/**
 * Product data for updates (all fields optional except id)
 */
export type UpdateProductRequest = Partial<Omit<Product, 
  | 'id' 
  | 'createdAt' 
  | 'updatedAt'
>> & {
  /** Product ID is required for updates */
  readonly id: string;
};

/**
 * Product filtering options
 */
export interface ProductFilters {
  /** Filter by category */
  category?: ProductCategory | ProductCategory[];
  
  /** Filter by status */
  status?: ProductStatus | ProductStatus[];
  
  /** Price range in cents */
  priceRange?: {
    min: number;
    max: number;
  };
  
  /** Minimum rating */
  minRating?: number;
  
  /** Search term for name/description */
  search?: string;
  
  /** Whether to include only featured products */
  featuredOnly?: boolean;
  
  /** Whether to include only in-stock products */
  inStockOnly?: boolean;
}

/**
 * Product sorting options
 */
export interface ProductSort {
  /** Field to sort by */
  field: 'name' | 'priceInCents' | 'averageRating' | 'createdAt' | 'updatedAt';
  
  /** Sort direction */
  direction: 'asc' | 'desc';
}

/**
 * Paginated product list response
 */
export interface ProductList {
  /** Array of products */
  products: ProductSummary[];
  
  /** Pagination information */
  pagination: {
    /** Current page (1-based) */
    page: number;
    /** Items per page */
    limit: number;
    /** Total number of products */
    total: number;
    /** Total number of pages */
    pages: number;
    /** Whether there's a next page */
    hasNext: boolean;
    /** Whether there's a previous page */
    hasPrev: boolean;
  };
  
  /** Applied filters */
  filters: ProductFilters;
  
  /** Applied sorting */
  sort: ProductSort;
}

/**
 * Utility type for product price calculations
 */
export interface ProductPriceInfo {
  /** Formatted price string (e.g., "$199.99") */
  formattedPrice: string;
  
  /** Price including tax */
  priceWithTax: number;
  
  /** Tax amount */
  taxAmount: number;
  
  /** Whether product is on sale */
  onSale: boolean;
  
  /** Discount percentage if on sale */
  discountPercentage?: number;
  
  /** Savings amount if on sale */
  savings?: number;
}

/**
 * Type guard to check if a product is available for purchase
 */
export const isProductAvailable = (product: Product): boolean => {
  return product.status === ProductStatus.AVAILABLE && 
         product.inventory.quantity > 0;
};

/**
 * Type guard to check if a product is on sale
 */
export const isProductOnSale = (product: Product): boolean => {
  return product.pricing.originalPriceInCents !== undefined &&
         product.pricing.originalPriceInCents > product.pricing.priceInCents;
};

/**
 * Calculate product price information with tax and discounts
 */
export const calculateProductPrice = (product: Product): ProductPriceInfo => {
  const { priceInCents, originalPriceInCents, taxRate, currency } = product.pricing;
  
  const priceWithTax = Math.round(priceInCents * (1 + taxRate));
  const taxAmount = priceWithTax - priceInCents;
  const onSale = isProductOnSale(product);
  
  let discountPercentage: number | undefined;
  let savings: number | undefined;
  
  if (onSale && originalPriceInCents) {
    savings = originalPriceInCents - priceInCents;
    discountPercentage = Math.round((savings / originalPriceInCents) * 100);
  }
  
  return {
    formattedPrice: formatCurrency(priceInCents, currency),
    priceWithTax,
    taxAmount,
    onSale,
    discountPercentage,
    savings
  };
};

/**
 * Format price in cents to currency string
 */
const formatCurrency = (priceInCents: number, currency: string): string => {
  const price = priceInCents / 100;
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(price);
};
```

### 2. Hook Documentation

```typescript
/**
 * @fileoverview Shopping cart management hook
 * 
 * Provides cart state management with persistence, validation,
 * and optimistic updates for a seamless user experience.
 */

/**
 * Cart item with quantity information
 */
export interface CartItem {
  /** Product information */
  product: ProductSummary;
  
  /** Quantity of this product in cart */
  quantity: number;
  
  /** When this item was added to cart */
  addedAt: Date;
}

/**
 * Shopping cart state and actions
 */
export interface CartState {
  /** All items in the cart */
  items: CartItem[];
  
  /** Total number of items */
  totalItems: number;
  
  /** Total price in cents (before tax) */
  totalPrice: number;
  
  /** Whether cart is being synced with server */
  isLoading: boolean;
  
  /** Last sync error if any */
  error: string | null;
}

/**
 * Shopping cart management hook
 * 
 * Provides complete cart functionality including:
 * - Adding/removing/updating items
 * - Persistent storage (localStorage + server sync for authenticated users)
 * - Optimistic updates for better UX
 * - Automatic inventory validation
 * - Cart abandonment recovery
 * 
 * @example
 * ```typescript
 * function ProductCard({ product }: { product: ProductSummary }) {
 *   const {
 *     items,
 *     totalItems,
 *     addItem,
 *     removeItem,
 *     updateQuantity,
 *     isInCart,
 *     getItemQuantity
 *   } = useShoppingCart();
 *   
 *   const handleAddToCart = () => {
 *     addItem(product, 1);
 *   };
 *   
 *   const currentQuantity = getItemQuantity(product.id);
 *   
 *   return (
 *     <div>
 *       <h3>{product.name}</h3>
 *       {isInCart(product.id) ? (
 *         <div>
 *           <button onClick={() => updateQuantity(product.id, currentQuantity - 1)}>
 *             -
 *           </button>
 *           <span>{currentQuantity}</span>
 *           <button onClick={() => updateQuantity(product.id, currentQuantity + 1)}>
 *             +
 *           </button>
 *         </div>
 *       ) : (
 *         <button onClick={handleAddToCart}>
 *           Add to Cart
 *         </button>
 *       )}
 *     </div>
 *   );
 * }
 * ```
 * 
 * @example
 * ```typescript
 * function CartSummary() {
 *   const { items, totalItems, totalPrice, clearCart, isLoading } = useShoppingCart();
 *   
 *   if (items.length === 0) {
 *     return <div>Your cart is empty</div>;
 *   }
 *   
 *   return (
 *     <div>
 *       <h2>Cart ({totalItems} items)</h2>
 *       <div>Total: ${(totalPrice / 100).toFixed(2)}</div>
 *       <button onClick={clearCart} disabled={isLoading}>
 *         Clear Cart
 *       </button>
 *     </div>
 *   );
 * }
 * ```
 * 
 * @returns Cart state and management functions
 */
export const useShoppingCart = () => {
  // State management
  const [state, setState] = useState<CartState>({
    items: [],
    totalItems: 0,
    totalPrice: 0,
    isLoading: false,
    error: null
  });
  
  const { user } = useAuth();
  const queryClient = useQueryClient();
  
  // Load cart from storage on mount
  useEffect(() => {
    loadCartFromStorage();
  }, []);
  
  // Sync with server for authenticated users
  useEffect(() => {
    if (user) {
      syncCartWithServer();
    }
  }, [user]);
  
  // Persist to localStorage whenever cart changes
  useEffect(() => {
    if (state.items.length > 0) {
      persistCartToStorage(state.items);
    }
  }, [state.items]);
  
  /**
   * Load cart from localStorage
   * 
   * Validates stored items and removes any that are no longer valid
   * (e.g., products that no longer exist or are out of stock).
   */
  const loadCartFromStorage = async () => {
    try {
      const storedCart = localStorage.getItem('shopping-cart');
      if (!storedCart) return;
      
      const cartItems: CartItem[] = JSON.parse(storedCart);
      
      // Validate cart items against current product data
      const validatedItems = await validateCartItems(cartItems);
      
      updateCartState(validatedItems);
    } catch (error) {
      console.error('Failed to load cart from storage:', error);
      localStorage.removeItem('shopping-cart');
    }
  };
  
  /**
   * Validate cart items against current product information
   * 
   * Removes items that are no longer available and updates
   * quantities that exceed current stock levels.
   * 
   * @param items - Cart items to validate
   * @returns Promise resolving to validated cart items
   */
  const validateCartItems = async (items: CartItem[]): Promise<CartItem[]> => {
    const productIds = items.map(item => item.product.id);
    
    try {
      // Fetch current product data
      const products = await queryClient.fetchQuery(
        ['products', 'validate', productIds],
        () => productApi.getProductsByIds(productIds)
      );
      
      const productMap = new Map(products.map(p => [p.id, p]));
      
      return items
        .map(item => {
          const currentProduct = productMap.get(item.product.id);
          if (!currentProduct) return null; // Product no longer exists
          
          if (!isProductAvailable(currentProduct)) return null; // Product not available
          
          // Update quantity if it exceeds current stock
          const maxQuantity = Math.min(item.quantity, currentProduct.inventory.quantity);
          
          return {
            ...item,
            product: {
              ...item.product,
              ...currentProduct // Update with latest product data
            },
            quantity: maxQuantity
          };
        })
        .filter((item): item is CartItem => item !== null);
    } catch (error) {
      console.error('Cart validation failed:', error);
      return items; // Return original items if validation fails
    }
  };
  
  /**
   * Sync cart with server for authenticated users
   * 
   * Merges local cart with server cart, preferring more recent items.
   */
  const syncCartWithServer = async () => {
    if (!user) return;
    
    setState(prev => ({ ...prev, isLoading: true, error: null }));
    
    try {
      const serverCart = await cartApi.getCart();
      const mergedItems = mergeCartItems(state.items, serverCart.items);
      
      // Update server with merged cart
      await cartApi.updateCart(mergedItems);
      
      updateCartState(mergedItems);
    } catch (error) {
      console.error('Cart sync failed:', error);
      setState(prev => ({ 
        ...prev, 
        isLoading: false, 
        error: 'Failed to sync cart. Changes may not be saved.' 
      }));
    }
  };
  
  /**
   * Merge local and server cart items
   * 
   * For duplicate products, keeps the item with the most recent addedAt timestamp
   * and combines quantities up to the available stock limit.
   * 
   * @param localItems - Items from local storage
   * @param serverItems - Items from server
   * @returns Merged cart items
   */
  const mergeCartItems = (localItems: CartItem[], serverItems: CartItem[]): CartItem[] => {
    const itemMap = new Map<string, CartItem>();
    
    // Add server items first
    serverItems.forEach(item => {
      itemMap.set(item.product.id, item);
    });
    
    // Merge with local items
    localItems.forEach(localItem => {
      const serverItem = itemMap.get(localItem.product.id);
      
      if (serverItem) {
        // Product exists in both - merge quantities and use most recent data
        const isLocalNewer = new Date(localItem.addedAt) > new Date(serverItem.addedAt);
        const maxQuantity = Math.min(
          localItem.quantity + serverItem.quantity,
          localItem.product.inventory?.quantity || 999
        );
        
        itemMap.set(localItem.product.id, {
          product: isLocalNewer ? localItem.product : serverItem.product,
          quantity: maxQuantity,
          addedAt: isLocalNewer ? localItem.addedAt : serverItem.addedAt
        });
      } else {
        // Local item only
        itemMap.set(localItem.product.id, localItem);
      }
    });
    
    return Array.from(itemMap.values());
  };
  
  /**
   * Update cart state with new items
   * 
   * Recalculates totals and updates the state atomically.
   * 
   * @param items - New cart items
   */
  const updateCartState = (items: CartItem[]) => {
    const totalItems = items.reduce((sum, item) => sum + item.quantity, 0);
    const totalPrice = items.reduce((sum, item) => sum + (item.product.priceInCents * item.quantity), 0);
    
    setState(prev => ({
      ...prev,
      items,
      totalItems,
      totalPrice,
      isLoading: false,
      error: null
    }));
  };
  
  /**
   * Add product to cart
   * 
   * If product already exists, increases quantity by the specified amount.
   * Validates that the total quantity doesn't exceed available stock.
   * 
   * @param product - Product to add
   * @param quantity - Quantity to add (default: 1)
   * @throws {Error} When quantity exceeds available stock
   * 
   * @example
   * ```typescript
   * const { addItem } = useShoppingCart();
   * 
   * try {
   *   await addItem(product, 2);
   *   toast.success('Added to cart');
   * } catch (error) {
   *   toast.error(error.message);
   * }
   * ```
   */
  const addItem = useCallback(async (product: ProductSummary, quantity: number = 1) => {
    // Validation
    if (quantity <= 0) {
      throw new Error('Quantity must be greater than 0');
    }
    
    if (!isProductAvailable(product)) {
      throw new Error('Product is not available');
    }
    
    // Check stock availability
    const existingItem = state.items.find(item => item.product.id === product.id);
    const currentQuantity = existingItem?.quantity || 0;
    const newTotalQuantity = currentQuantity + quantity;
    
    if (product.inventory && newTotalQuantity > product.inventory.quantity) {
      throw new Error(`Only ${product.inventory.quantity} items available`);
    }
    
    // Optimistic update
    const now = new Date();
    let newItems: CartItem[];
    
    if (existingItem) {
      // Update existing item
      newItems = state.items.map(item =>
        item.product.id === product.id
          ? { ...item, quantity: newTotalQuantity, addedAt: now }
          : item
      );
    } else {
      // Add new item
      newItems = [...state.items, {
        product,
        quantity,
        addedAt: now
      }];
    }
    
    updateCartState(newItems);
    
    // Sync with server if user is authenticated
    if (user) {
      try {
        await cartApi.addItem(product.id, quantity);
      } catch (error) {
        console.error('Failed to sync cart addition:', error);
        // Don't revert optimistic update for better UX
      }
    }
    
    // Track analytics
    trackEvent('add_to_cart', {
      product_id: product.id,
      product_name: product.name,
      quantity,
      price: product.priceInCents
    });
  }, [state.items, user]);
  
  /**
   * Remove product from cart completely
   * 
   * @param productId - ID of product to remove
   * 
   * @example
   * ```typescript
   * const { removeItem } = useShoppingCart();
   * 
   * const handleRemove = () => {
   *   removeItem(product.id);
   *   toast.success('Removed from cart');
   * };
   * ```
   */
  const removeItem = useCallback(async (productId: string) => {
    const itemToRemove = state.items.find(item => item.product.id === productId);
    if (!itemToRemove) return;
    
    // Optimistic update
    const newItems = state.items.filter(item => item.product.id !== productId);
    updateCartState(newItems);
    
    // Sync with server
    if (user) {
      try {
        await cartApi.removeItem(productId);
      } catch (error) {
        console.error('Failed to sync cart removal:', error);
      }
    }
    
    // Track analytics
    trackEvent('remove_from_cart', {
      product_id: productId,
      product_name: itemToRemove.product.name,
      quantity: itemToRemove.quantity
    });
  }, [state.items, user]);
  
  /**
   * Update quantity of specific product in cart
   * 
   * Sets the absolute quantity, not relative. Use 0 to remove the item.
   * 
   * @param productId - ID of product to update
   * @param newQuantity - New absolute quantity (0 to remove)
   * @throws {Error} When quantity exceeds available stock
   * 
   * @example
   * ```typescript
   * const { updateQuantity } = useShoppingCart();
   * 
   * const increaseQuantity = () => {
   *   const currentQty = getItemQuantity(product.id);
   *   updateQuantity(product.id, currentQty + 1);
   * };
   * ```
   */
  const updateQuantity = useCallback(async (productId: string, newQuantity: number) => {
    if (newQuantity < 0) {
      throw new Error('Quantity cannot be negative');
    }
    
    if (newQuantity === 0) {
      return removeItem(productId);
    }
    
    const existingItem = state.items.find(item => item.product.id === productId);
    if (!existingItem) {
      throw new Error('Item not found in cart');
    }
    
    // Validate stock
    if (existingItem.product.inventory && newQuantity > existingItem.product.inventory.quantity) {
      throw new Error(`Only ${existingItem.product.inventory.quantity} items available`);
    }
    
    // Optimistic update
    const newItems = state.items.map(item =>
      item.product.id === productId
        ? { ...item, quantity: newQuantity }
        : item
    );
    
    updateCartState(newItems);
    
    // Sync with server
    if (user) {
      try {
        await cartApi.updateItemQuantity(productId, newQuantity);
      } catch (error) {
        console.error('Failed to sync quantity update:', error);
      }
    }
  }, [state.items, user, removeItem]);
  
  /**
   * Clear all items from cart
   * 
   * @example
   * ```typescript
   * const { clearCart } = useShoppingCart();
   * 
   * const handleClearCart = async () => {
   *   if (confirm('Remove all items from cart?')) {
   *     await clearCart();
   *   }
   * };
   * ```
   */
  const clearCart = useCallback(async () => {
    // Optimistic update
    updateCartState([]);
    
    // Clear local storage
    localStorage.removeItem('shopping-cart');
    
    // Sync with server
    if (user) {
      try {
        await cartApi.clearCart();
      } catch (error) {
        console.error('Failed to sync cart clear:', error);
      }
    }
    
    // Track analytics
    trackEvent('clear_cart', {
      items_count: state.items.length,
      total_value: state.totalPrice
    });
  }, [state.items.length, state.totalPrice, user]);
  
  /**
   * Check if product is in cart
   * 
   * @param productId - Product ID to check
   * @returns True if product is in cart
   */
  const isInCart = useCallback((productId: string): boolean => {
    return state.items.some(item => item.product.id === productId);
  }, [state.items]);
  
  /**
   * Get quantity of specific product in cart
   * 
   * @param productId - Product ID to check
   * @returns Current quantity in cart (0 if not in cart)
   */
  const getItemQuantity = useCallback((productId: string): number => {
    const item = state.items.find(item => item.product.id === productId);
    return item?.quantity || 0;
  }, [state.items]);
  
  /**
   * Get cart item by product ID
   * 
   * @param productId - Product ID to find
   * @returns Cart item if found, undefined otherwise
   */
  const getCartItem = useCallback((productId: string): CartItem | undefined => {
    return state.items.find(item => item.product.id === productId);
  }, [state.items]);
  
  /**
   * Persist cart to localStorage
   * 
   * @param items - Cart items to persist
   */
  const persistCartToStorage = (items: CartItem[]) => {
    try {
      localStorage.setItem('shopping-cart', JSON.stringify(items));
    } catch (error) {
      console.error('Failed to persist cart to storage:', error);
    }
  };
  
  return {
    // State
    items: state.items,
    totalItems: state.totalItems,
    totalPrice: state.totalPrice,
    isLoading: state.isLoading,
    error: state.error,
    
    // Actions
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    
    // Queries
    isInCart,
    getItemQuantity,
    getCartItem,
    
    // Utils
    isEmpty: state.items.length === 0,
    hasItems: state.items.length > 0
  };
};
```

## 🔄 Handoff Processes

### 1. QA Handoff Documentation

```markdown
# QA Handoff: Product Search Feature

## Feature Overview
Implementation of advanced product search with filtering, sorting, and pagination capabilities.

## Development Summary
- **Epic**: [E-123] Advanced Search Implementation
- **Stories**: [S-456] Search UI, [S-457] Filter System, [S-458] Search API
- **Developer**: @john.doe
- **Completion Date**: 2024-01-15
- **Branch**: `feature/advanced-search`

## Testing Requirements

### Functional Testing
- [ ] **Search Functionality**
  - [ ] Search by product name
  - [ ] Search by description
  - [ ] Search by SKU
  - [ ] Empty search handling
  - [ ] Special character handling
  - [ ] Unicode support

- [ ] **Filtering System**
  - [ ] Category filtering (single/multiple)
  - [ ] Price range filtering
  - [ ] Rating filtering
  - [ ] Availability filtering
  - [ ] Brand filtering
  - [ ] Filter combinations
  - [ ] Filter clearing

- [ ] **Sorting Options**
  - [ ] Sort by relevance (default)
  - [ ] Sort by price (low to high)
  - [ ] Sort by price (high to low)
  - [ ] Sort by rating
  - [ ] Sort by newest first
  - [ ] Sort by popularity

- [ ] **Pagination**
  - [ ] Page navigation
  - [ ] Items per page selection
  - [ ] URL state preservation
  - [ ] Performance with large datasets

### Performance Testing
- [ ] **Search Performance**
  - [ ] Search response time < 500ms
  - [ ] Filter response time < 200ms
  - [ ] Page load time < 2s
  - [ ] Bundle size impact < 50KB

- [ ] **Load Testing**
  - [ ] Concurrent user search (100 users)
  - [ ] Large result sets (1000+ products)
  - [ ] Filter combinations performance
  - [ ] Memory usage during search

### Accessibility Testing
- [ ] **Keyboard Navigation**
  - [ ] Tab order logical
  - [ ] Search input accessible via keyboard
  - [ ] Filter controls keyboard accessible
  - [ ] Pagination keyboard accessible

- [ ] **Screen Reader Support**
  - [ ] Search results announced
  - [ ] Filter changes announced
  - [ ] Loading states announced
  - [ ] Error states accessible

- [ ] **Visual Requirements**
  - [ ] Color contrast meets WCAG AA
  - [ ] Focus indicators visible
  - [ ] Text scales to 200%
  - [ ] Works with high contrast mode

### Browser Compatibility
- [ ] **Desktop Browsers**
  - [ ] Chrome 100+ ✓
  - [ ] Firefox 98+ ✓
  - [ ] Safari 15+ ✓
  - [ ] Edge 100+ ✓

- [ ] **Mobile Browsers**
  - [ ] iOS Safari 15+ ✓
  - [ ] Chrome Mobile 100+ ✓
  - [ ] Samsung Internet 16+ ✓

### Error Handling
- [ ] **Network Errors**
  - [ ] Search API failure
  - [ ] Timeout handling
  - [ ] Offline detection
  - [ ] Retry mechanisms

- [ ] **User Input Errors**
  - [ ] Invalid characters
  - [ ] SQL injection attempts
  - [ ] XSS attempt handling
  - [ ] Rate limiting responses

## Test Data

### Sample Search Queries
```
- "wireless headphones"
- "laptop 16gb"
- "nike shoes size 10"
- "€ $ £ ¥" (special characters)
- "café résumé" (unicode)
- "" (empty search)
- "   " (whitespace only)
```

### Test Products
- Products in all categories
- Products with varying price ranges ($1 - $5000)
- Products with different ratings (1-5 stars)
- Products with different availability states
- Products with long/short names and descriptions

## Known Issues & Limitations
1. **Search Relevance**: Relevance algorithm needs refinement based on user feedback
2. **Real-time Filters**: Filters update search results immediately (may cause performance issues with slow connections)
3. **Search History**: Not implemented in this phase
4. **Saved Searches**: Not implemented in this phase

## API Documentation

### Search Endpoint
```
GET /api/products/search
Query Parameters:
- q: string (search query)
- category: string[] (category filters)
- priceMin: number (minimum price in cents)
- priceMax: number (maximum price in cents)
- minRating: number (minimum rating)
- sortBy: string (sort field)
- sortOrder: 'asc' | 'desc'
- page: number (page number, 1-based)
- limit: number (items per page, max 100)

Response:
{
  "products": [...],
  "pagination": {...},
  "filters": {...},
  "facets": {...}
}
```

## Performance Benchmarks
- **Search Response Time**: Target < 500ms, Current avg: 380ms
- **Bundle Size Impact**: Target < 50KB, Actual: 42KB gzipped
- **Core Web Vitals**: All green on test pages
- **Lighthouse Score**: 95+ for all metrics

## Deployment Notes
- Feature flagged: `ENABLE_ADVANCED_SEARCH=true`
- Requires search index rebuild (handled automatically)
- No database migrations required
- CDN cache invalidation needed for static assets

## Rollback Plan
1. Set feature flag `ENABLE_ADVANCED_SEARCH=false`
2. Deploy previous search component version
3. Clear CDN cache
4. Monitor error rates and user feedback

## QA Sign-off

### Testing Checklist
- [ ] All functional requirements tested
- [ ] Performance requirements verified
- [ ] Accessibility requirements met
- [ ] Browser compatibility confirmed
- [ ] Error scenarios handled
- [ ] Mobile responsiveness verified

### QA Engineer: _________________ Date: _________
### Test Lead Approval: _________________ Date: _________

## Additional Notes
- Consider A/B testing the relevance algorithm
- Monitor search queries for insights into user behavior
- Plan to implement search analytics in next iteration
```

### 2. DevOps Handoff Documentation

```markdown
# DevOps Handoff: Product Search Feature Deployment

## Deployment Summary
- **Feature**: Advanced Product Search
- **Release Version**: v2.1.0
- **Target Environments**: Staging → Production
- **Deployment Window**: 2024-01-20 02:00 UTC (Saturday)
- **Rollback Window**: 4 hours

## Infrastructure Requirements

### Environment Variables
```bash
# Search Configuration
SEARCH_SERVICE_URL=https://search-api.internal.com
SEARCH_INDEX_NAME=products_v2
SEARCH_API_KEY=****
ELASTICSEARCH_URL=https://elasticsearch.internal.com:9200
ELASTICSEARCH_USERNAME=search_user
ELASTICSEARCH_PASSWORD=****

# Feature Flags
ENABLE_ADVANCED_SEARCH=false  # Initially disabled
ENABLE_SEARCH_ANALYTICS=true
SEARCH_CACHE_TTL=300

# Performance Settings
SEARCH_REQUEST_TIMEOUT=5000
SEARCH_MAX_RESULTS=1000
SEARCH_RATE_LIMIT=100  # requests per minute per IP
```

### New Dependencies
```json
{
  "elasticsearch": "^7.17.0",
  "search-insights": "^2.1.0",
  "fuse.js": "^6.6.0"
}
```

### Resource Requirements
- **CPU**: +10% during search operations
- **Memory**: +50MB for search cache
- **Storage**: +2GB for search indices
- **Network**: Search service connectivity required

## Database Changes

### Migrations
```sql
-- Add search analytics table
CREATE TABLE search_analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    query TEXT NOT NULL,
    user_id UUID REFERENCES users(id),
    session_id VARCHAR(255),
    results_count INTEGER,
    clicked_result_id UUID,
    search_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Add search index for products
CREATE INDEX CONCURRENTLY idx_products_search_vector 
ON products USING gin(to_tsvector('english', name || ' ' || description));

-- Add performance index
CREATE INDEX CONCURRENTLY idx_products_category_price 
ON products(category, price_in_cents) 
WHERE status = 'available';
```

### Index Rebuild
```bash
# Rebuild search index (run during maintenance window)
curl -X POST "localhost:9200/products_v2/_reindex" \
  -H "Content-Type: application/json" \
  -d '{
    "source": {
      "index": "products_v1"
    },
    "dest": {
      "index": "products_v2"
    }
  }'
```

## Deployment Steps

### Pre-deployment
1. **Health Checks**
   ```bash
   # Verify search service connectivity
   curl -f https://search-api.internal.com/health
   
   # Verify Elasticsearch cluster health
   curl -f https://elasticsearch.internal.com:9200/_cluster/health
   
   # Check database connectivity
   psql -h db.internal.com -U app_user -c "SELECT 1"
   ```

2. **Backup Creation**
   ```bash
   # Database backup
   pg_dump -h prod-db.com -U backup_user app_db > backup_$(date +%Y%m%d_%H%M%S).sql
   
   # Search index backup
   curl -X PUT "elasticsearch:9200/_snapshot/backup/products_$(date +%Y%m%d_%H%M%S)"
   ```

3. **Traffic Preparation**
   ```bash
   # Scale up instances before deployment
   kubectl scale deployment app --replicas=6
   
   # Warm up CDN cache
   curl -X POST https://api.cloudflare.com/client/v4/zones/$ZONE_ID/purge_cache \
     -H "Authorization: Bearer $CF_TOKEN" \
     -d '{"purge_everything":true}'
   ```

### Deployment Sequence

#### Stage 1: Infrastructure (30 minutes)
```bash
# 1. Deploy search service updates
kubectl apply -f k8s/search-service.yaml

# 2. Update environment variables
kubectl create configmap app-config --from-env-file=.env.production

# 3. Run database migrations
npm run db:migrate:production

# 4. Rebuild search indices
npm run search:reindex:production

# 5. Verify infrastructure
./scripts/verify-infrastructure.sh
```

#### Stage 2: Application Deployment (45 minutes)
```bash
# 1. Build and push Docker image
docker build -t app:v2.1.0 .
docker push registry.internal.com/app:v2.1.0

# 2. Rolling deployment (zero downtime)
kubectl set image deployment/app app=registry.internal.com/app:v2.1.0

# 3. Wait for rollout completion
kubectl rollout status deployment/app --timeout=600s

# 4. Verify deployment
kubectl get pods -l app=main
./scripts/smoke-test.sh
```

#### Stage 3: Feature Activation (15 minutes)
```bash
# 1. Enable feature flag for 10% of users
curl -X POST https://feature-flags.internal.com/flags/ENABLE_ADVANCED_SEARCH \
  -d '{"enabled": true, "percentage": 10}'

# 2. Monitor metrics for 10 minutes
./scripts/monitor-metrics.sh --duration=600

# 3. Gradually increase to 100%
for percentage in 25 50 75 100; do
  curl -X PUT https://feature-flags.internal.com/flags/ENABLE_ADVANCED_SEARCH \
    -d "{\"percentage\": $percentage}"
  sleep 300  # Wait 5 minutes between increases
done
```

## Monitoring & Alerting

### Key Metrics
```yaml
# Search Performance
- search_response_time_p95 < 500ms
- search_error_rate < 1%
- search_requests_per_second

# Infrastructure
- elasticsearch_cluster_health = green
- search_service_uptime > 99.9%
- database_connection_pool_usage < 80%

# Business Metrics
- search_conversion_rate
- search_abandonment_rate
- average_search_results_clicked
```

### Alerts Configuration
```yaml
alerts:
  - name: "Search Response Time High"
    condition: "search_response_time_p95 > 1000ms for 5 minutes"
    severity: "warning"
    channels: ["#ops-alerts", "#search-team"]
    
  - name: "Search Service Down"
    condition: "search_service_uptime < 99% for 2 minutes"
    severity: "critical"
    channels: ["#ops-critical", "#search-team", "pager-duty"]
    
  - name: "Elasticsearch Cluster Red"
    condition: "elasticsearch_cluster_health = red for 1 minute"
    severity: "critical"
    channels: ["#ops-critical", "pager-duty"]
```

### Dashboard URLs
- **Application Metrics**: https://grafana.internal.com/d/app-metrics
- **Search Analytics**: https://grafana.internal.com/d/search-analytics
- **Infrastructure Health**: https://grafana.internal.com/d/infrastructure
- **Business KPIs**: https://grafana.internal.com/d/business-kpis

## Rollback Procedures

### Automatic Rollback Triggers
- Error rate > 5% for 5 minutes
- Response time > 2000ms for 10 minutes
- Search service unavailable for 2 minutes
- Elasticsearch cluster health = red for 5 minutes

### Manual Rollback Steps
```bash
# 1. Disable feature flag immediately
curl -X PUT https://feature-flags.internal.com/flags/ENABLE_ADVANCED_SEARCH \
  -d '{"enabled": false}'

# 2. Rollback application deployment
kubectl rollout undo deployment/app

# 3. Verify rollback
kubectl rollout status deployment/app
./scripts/smoke-test.sh

# 4. Restore previous search index if needed
curl -X POST "elasticsearch:9200/_snapshot/backup/products_backup/_restore"

# 5. Notify stakeholders
./scripts/send-rollback-notification.sh
```

## Post-Deployment Verification

### Health Checks (First 2 hours)
```bash
# Run comprehensive health checks
./scripts/post-deployment-health-check.sh

# Verify search functionality
curl -f "https://api.myapp.com/products/search?q=test"

# Check performance metrics
./scripts/verify-performance-metrics.sh

# Validate feature flag system
curl -f "https://feature-flags.internal.com/flags/ENABLE_ADVANCED_SEARCH"
```

### Performance Validation
- [ ] Search response time < 500ms (p95)
- [ ] Page load time < 2s
- [ ] Error rate < 1%
- [ ] All monitoring dashboards green
- [ ] No memory leaks detected
- [ ] CDN cache hit rate > 80%

### Business Validation
- [ ] Search results quality verified by product team
- [ ] Analytics data flowing correctly
- [ ] User feedback collection active
- [ ] A/B test framework ready

## Support Information

### Runbooks
- **Search Service Issues**: https://wiki.internal.com/runbooks/search-service
- **Elasticsearch Operations**: https://wiki.internal.com/runbooks/elasticsearch
- **Feature Flag Management**: https://wiki.internal.com/runbooks/feature-flags

### Escalation Contacts
- **Primary On-Call**: @ops-team (Slack: #ops-oncall)
- **Search Team Lead**: @alice.smith (Phone: +1-555-0123)
- **DevOps Lead**: @bob.jones (Phone: +1-555-0456)
- **Incident Commander**: @incident-commander (Auto-assigned via PagerDuty)

### Log Locations
```bash
# Application logs
kubectl logs -f deployment/app -c app

# Search service logs
kubectl logs -f deployment/search-service

# Elasticsearch logs
kubectl logs -f statefulset/elasticsearch

# Load balancer logs
aws logs filter-log-events --log-group-name=/aws/elb/app-lb
```

## Success Criteria
- [ ] Zero downtime deployment completed
- [ ] All health checks passing
- [ ] Feature flag rollout to 100% successful
- [ ] Performance metrics within targets
- [ ] No critical alerts triggered
- [ ] Business metrics showing expected patterns
- [ ] Documentation updated
- [ ] Team notified of successful deployment

---

**Deployment Engineer**: _________________ Date: _________
**DevOps Lead Approval**: _________________ Date: _________
**Release Manager Approval**: _________________ Date: _________
```

## 📝 Exercise: Complete Documentation Package

Create a comprehensive documentation package for a feature in your application:

### Step 1: Code Documentation
- Add comprehensive JSDoc to components and hooks
- Create detailed type definitions with examples
- Document API services with request/response schemas
- Include inline comments for complex business logic

### Step 2: Architecture Documentation
- Write an ADR for a significant technical decision
- Create API documentation with examples
- Document component architecture and data flow
- Include performance considerations and trade-offs

### Step 3: Handoff Documentation
- Create QA handoff with test cases and acceptance criteria
- Write DevOps handoff with deployment procedures
- Include monitoring and alerting requirements
- Document rollback procedures and success criteria

### Step 4: Maintenance Documentation
- Create troubleshooting guides for common issues
- Document configuration and environment variables
- Include team contacts and escalation procedures
- Create runbooks for operational procedures

**Deliverable:** A complete documentation package that enables any team member to understand, test, deploy, and maintain the feature independently.

---

## 🔄 Modern Documentation Automation

### Automated Documentation Generation

```typescript
// scripts/generate-docs.ts
import { generateApiDocs } from './api-docs-generator';
import { generateComponentDocs } from './component-docs-generator';
import { generateTypeDocs } from './type-docs-generator';

/**
 * Automated documentation generation pipeline
 * 
 * Generates comprehensive documentation from code annotations,
 * TypeScript types, and API schemas.
 */
async function generateDocumentation() {
  console.log('🚀 Generating documentation...');
  
  try {
    // Generate API documentation from OpenAPI schemas
    await generateApiDocs({
      inputDir: './src/api',
      outputDir: './docs/api',
      format: 'markdown'
    });
    
    // Generate component documentation from TypeScript
    await generateComponentDocs({
      inputDir: './src/components',
      outputDir: './docs/components',
      includeProps: true,
      includeExamples: true,
      includeTests: true
    });
    
    // Generate type documentation
    await generateTypeDocs({
      inputDir: './src/types',
      outputDir: './docs/types',
      includeInheritance: true,
      includeExamples: true
    });
    
    console.log('✅ Documentation generated successfully');
  } catch (error) {
    console.error('❌ Documentation generation failed:', error);
    process.exit(1);
  }
}

if (require.main === module) {
  generateDocumentation();
}
```

### Living Documentation with Storybook

```typescript
// .storybook/main.ts
export default {
  stories: ['../src/**/*.stories.@(js|jsx|ts|tsx|mdx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-docs',
    '@storybook/addon-controls',
    '@storybook/addon-a11y',
    '@storybook/addon-design-tokens',
    'storybook-addon-pseudo-states'
  ],
  features: {
    buildStoriesJson: true,
    interactionsDebugger: true
  },
  typescript: {
    check: false,
    reactDocgen: 'react-docgen-typescript'
  }
};

// ProductCard.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { ProductCard } from './ProductCard';
import { mockProducts } from '../__mocks__/products';

const meta: Meta<typeof ProductCard> = {
  title: 'Components/ProductCard',
  component: ProductCard,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: `
ProductCard displays product information in a visually appealing card format.
It supports various states including loading, error, and different variants.

## Features
- Responsive design for mobile and desktop
- Accessibility support with ARIA labels
- Loading states and error handling
- Multiple visual variants
- Add to cart functionality
        `
      }
    }
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['default', 'compact', 'featured'],
      description: 'Visual variant of the card'
    },
    showAddToCart: {
      control: 'boolean',
      description: 'Whether to show the add to cart button'
    }
  }
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    product: mockProducts[0],
    onAddToCart: (product) => console.log('Added to cart:', product),
    variant: 'default',
    showAddToCart: true
  }
};

export const Compact: Story = {
  args: {
    ...Default.args,
    variant: 'compact'
  }
};

export const Featured: Story = {
  args: {
    ...Default.args,
    variant: 'featured'
  }
};

export const OutOfStock: Story = {
  args: {
    ...Default.args,
    product: {
      ...mockProducts[0],
      inStock: false
    }
  }
};

export const OnSale: Story = {
  args: {
    ...Default.args,
    product: {
      ...mockProducts[0],
      originalPrice: 299.99,
      price: 199.99
    }
  }
};

export const WithoutAddToCart: Story = {
  args: {
    ...Default.args,
    showAddToCart: false
  }
};

// Interactive story with state
export const Interactive: Story = {
  render: (args) => {
    const [cartItems, setCartItems] = useState<string[]>([]);
    
    const handleAddToCart = (product: Product) => {
      setCartItems(prev => [...prev, product.id]);
      console.log('Added to cart:', product);
    };
    
    return (
      <div>
        <ProductCard {...args} onAddToCart={handleAddToCart} />
        <div style={{ marginTop: '1rem', fontSize: '0.875rem', color: '#666' }}>
          Cart items: {cartItems.length}
        </div>
      </div>
    );
  },
  args: Default.args
};
```

### Documentation CI/CD Pipeline

```yaml
# .github/workflows/documentation.yml
name: Documentation

on:
  push:
    branches: [main, develop]
    paths: 
      - 'src/**'
      - 'docs/**'
      - '.storybook/**'
  pull_request:
    branches: [main]
    paths:
      - 'src/**'
      - 'docs/**'

jobs:
  generate-docs:
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
      
      - name: Generate API documentation
        run: pnpm run docs:api
      
      - name: Generate component documentation
        run: pnpm run docs:components
      
      - name: Build Storybook
        run: pnpm run storybook:build
      
      - name: Check documentation completeness
        run: pnpm run docs:lint
      
      - name: Upload documentation artifacts
        uses: actions/upload-artifact@v3
        with:
          name: documentation
          path: |
            docs/
            storybook-static/
  
  deploy-docs:
    if: github.ref == 'refs/heads/main'
    needs: generate-docs
    runs-on: ubuntu-latest
    steps:
      - name: Download documentation
        uses: actions/download-artifact@v3
        with:
          name: documentation
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs
      
      - name: Deploy Storybook
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./storybook-static
          destination_dir: storybook
  
  validate-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Validate markdown links
        uses: gaurav-nelson/github-action-markdown-link-check@v1
        with:
          use-quiet-mode: 'yes'
          use-verbose-mode: 'yes'
      
      - name: Check documentation coverage
        run: |
          # Check that all public APIs have documentation
          pnpm run docs:coverage-check
      
      - name: Spell check documentation
        uses: streetsidesoftware/cspell-action@v2
        with:
          files: 'docs/**/*.md'
          incremental_files_only: false
```

### Documentation Quality Gates

```typescript
// scripts/docs-quality-gate.ts
import { glob } from 'glob';
import { readFileSync } from 'fs';
import { parseTypeScript } from './typescript-parser';

interface DocCoverageReport {
  totalComponents: number;
  documentedComponents: number;
  totalFunctions: number;
  documentedFunctions: number;
  coverage: number;
  missingDocs: string[];
}

/**
 * Documentation quality gate checker
 * 
 * Ensures all public APIs have proper documentation
 */
async function checkDocumentationCoverage(): Promise<DocCoverageReport> {
  const sourceFiles = await glob('src/**/*.{ts,tsx}', {
    ignore: ['**/*.test.*', '**/*.spec.*', '**/*.stories.*']
  });
  
  let totalComponents = 0;
  let documentedComponents = 0;
  let totalFunctions = 0;
  let documentedFunctions = 0;
  const missingDocs: string[] = [];
  
  for (const file of sourceFiles) {
    const content = readFileSync(file, 'utf8');
    const ast = parseTypeScript(content, file);
    
    // Check React components
    const components = ast.getComponentExports();
    totalComponents += components.length;
    
    for (const component of components) {
      if (component.hasJSDocComment) {
        documentedComponents++;
      } else {
        missingDocs.push(`Component ${component.name} in ${file}`);
      }
    }
    
    // Check exported functions
    const functions = ast.getFunctionExports();
    totalFunctions += functions.length;
    
    for (const func of functions) {
      if (func.hasJSDocComment) {
        documentedFunctions++;
      } else {
        missingDocs.push(`Function ${func.name} in ${file}`);
      }
    }
  }
  
  const totalItems = totalComponents + totalFunctions;
  const documentedItems = documentedComponents + documentedFunctions;
  const coverage = totalItems > 0 ? (documentedItems / totalItems) * 100 : 100;
  
  return {
    totalComponents,
    documentedComponents,
    totalFunctions,
    documentedFunctions,
    coverage,
    missingDocs
  };
}

async function enforceDocumentationStandards() {
  console.log('📋 Checking documentation coverage...');
  
  const report = await checkDocumentationCoverage();
  
  console.log(`
📊 Documentation Coverage Report
================================
Components: ${report.documentedComponents}/${report.totalComponents}
Functions: ${report.documentedFunctions}/${report.totalFunctions}
Overall Coverage: ${report.coverage.toFixed(1)}%
  `);
  
  const requiredCoverage = 80; // 80% minimum coverage
  
  if (report.coverage < requiredCoverage) {
    console.error(`❌ Documentation coverage (${report.coverage.toFixed(1)}%) is below required ${requiredCoverage}%`);
    
    if (report.missingDocs.length > 0) {
      console.error('\n🔍 Missing documentation for:');
      report.missingDocs.forEach(item => console.error(`  - ${item}`));
    }
    
    process.exit(1);
  }
  
  console.log(`✅ Documentation coverage meets requirements (${requiredCoverage}%)`);
}

if (require.main === module) {
  enforceDocumentationStandards();
}
```

### Interactive Documentation with Docusaurus

```typescript
// docusaurus.config.js
module.exports = {
  title: 'Project Documentation',
  tagline: 'Comprehensive documentation for our application',
  url: 'https://docs.myapp.com',
  baseUrl: '/',
  
  organizationName: 'mycompany',
  projectName: 'myapp-docs',
  
  presets: [
    [
      'classic',
      {
        docs: {
          sidebarPath: require.resolve('./sidebars.js'),
          editUrl: 'https://github.com/mycompany/myapp/tree/main/docs/',
          showLastUpdateAuthor: true,
          showLastUpdateTime: true,
        },
        blog: {
          showReadingTime: true,
          editUrl: 'https://github.com/mycompany/myapp/tree/main/docs/',
        },
        theme: {
          customCss: require.resolve('./src/css/custom.css'),
        },
      },
    ],
  ],
  
  themeConfig: {
    navbar: {
      title: 'MyApp Docs',
      logo: {
        alt: 'MyApp Logo',
        src: 'img/logo.svg',
      },
      items: [
        {
          type: 'doc',
          docId: 'getting-started',
          position: 'left',
          label: 'Docs',
        },
        {
          to: '/api',
          label: 'API',
          position: 'left',
        },
        {
          to: '/storybook',
          label: 'Components',
          position: 'left',
        },
        {
          href: 'https://github.com/mycompany/myapp',
          label: 'GitHub',
          position: 'right',
        },
      ],
    },
    
    footer: {
      style: 'dark',
      links: [
        {
          title: 'Documentation',
          items: [
            {
              label: 'Getting Started',
              to: '/docs/getting-started',
            },
            {
              label: 'API Reference',
              to: '/api',
            },
            {
              label: 'Components',
              to: '/storybook',
            },
          ],
        },
        {
          title: 'Community',
          items: [
            {
              label: 'Stack Overflow',
              href: 'https://stackoverflow.com/questions/tagged/myapp',
            },
            {
              label: 'Discord',
              href: 'https://discordapp.com/invite/myapp',
            },
          ],
        },
        {
          title: 'More',
          items: [
            {
              label: 'Blog',
              to: '/blog',
            },
            {
              label: 'GitHub',
              href: 'https://github.com/mycompany/myapp',
            },
          ],
        },
      ],
      copyright: `Copyright © ${new Date().getFullYear()} MyCompany. Built with Docusaurus.`,
    },
    
    prism: {
      theme: require('prism-react-renderer/themes/github'),
      darkTheme: require('prism-react-renderer/themes/dracula'),
      additionalLanguages: ['typescript', 'bash', 'yaml', 'json'],
    },
    
    algolia: {
      appId: 'YOUR_APP_ID',
      apiKey: 'YOUR_SEARCH_API_KEY',
      indexName: 'myapp-docs',
      contextualSearch: true,
    },
  },
  
  plugins: [
    [
      '@docusaurus/plugin-content-docs',
      {
        id: 'api',
        path: 'api',
        routeBasePath: 'api',
        sidebarPath: require.resolve('./sidebars-api.js'),
      },
    ],
  ],
};
```

## 📊 Documentation Analytics and Insights

```typescript
// Documentation analytics integration
class DocumentationAnalytics {
  private analytics: any;
  
  constructor() {
    // Initialize analytics (Google Analytics, Mixpanel, etc.)
    this.analytics = window.gtag || window.mixpanel;
  }
  
  trackDocumentationUsage(page: string, section: string) {
    this.analytics?.('event', 'documentation_view', {
      page,
      section,
      timestamp: Date.now()
    });
  }
  
  trackSearchQuery(query: string, resultsCount: number) {
    this.analytics?.('event', 'documentation_search', {
      query,
      results_count: resultsCount,
      timestamp: Date.now()
    });
  }
  
  trackFeedback(page: string, rating: number, comment?: string) {
    this.analytics?.('event', 'documentation_feedback', {
      page,
      rating,
      comment,
      timestamp: Date.now()
    });
  }
}

// Documentation feedback component
const DocumentationFeedback: React.FC<{ page: string }> = ({ page }) => {
  const [rating, setRating] = useState<number | null>(null);
  const [comment, setComment] = useState('');
  const [submitted, setSubmitted] = useState(false);
  
  const analytics = new DocumentationAnalytics();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (rating === null) return;
    
    try {
      await fetch('/api/documentation/feedback', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ page, rating, comment })
      });
      
      analytics.trackFeedback(page, rating, comment);
      setSubmitted(true);
    } catch (error) {
      console.error('Failed to submit feedback:', error);
    }
  };
  
  if (submitted) {
    return (
      <div className="feedback-success">
        <h3>Thank you for your feedback!</h3>
        <p>Your input helps us improve our documentation.</p>
      </div>
    );
  }
  
  return (
    <div className="documentation-feedback">
      <h3>Was this page helpful?</h3>
      <form onSubmit={handleSubmit}>
        <div className="rating-section">
          {[1, 2, 3, 4, 5].map((value) => (
            <button
              key={value}
              type="button"
              className={`rating-button ${rating === value ? 'selected' : ''}`}
              onClick={() => setRating(value)}
              aria-label={`Rate ${value} out of 5 stars`}
            >
              ⭐
            </button>
          ))}
        </div>
        
        <textarea
          placeholder="Tell us how we can improve this documentation..."
          value={comment}
          onChange={(e) => setComment(e.target.value)}
          rows={3}
        />
        
        <button type="submit" disabled={rating === null}>
          Submit Feedback
        </button>
      </form>
    </div>
  );
};
```

---

**Next:** [Module 9: Team & Workflow Alignment](../09-team-workflow/) - Master agile DoD, PR practices, branching strategies, and CI/CD integration.