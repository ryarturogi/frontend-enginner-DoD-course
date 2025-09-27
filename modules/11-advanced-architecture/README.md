# Module 11: Advanced Architecture & Code Organization

## 🎯 Learning Objectives

By the end of this module, you will:
- Master feature-based architecture that scales with team growth
- Understand when to choose monorepo vs polyrepo strategies
- Implement microfrontends with Module Federation when appropriate
- Design clean separation between UI, business logic, and data layers
- Choose and implement scalable state management patterns
- Create architecture that supports both current needs and future growth

## 🏗️ Feature-Based Architecture

### Why Feature-Based Architecture Matters

Traditional folder-by-type structures break down as applications grow:

```
❌ Folder-by-Type (doesn't scale)
src/
├── components/           # 50+ components mixed together
├── hooks/               # 30+ hooks with unclear ownership
├── services/            # 20+ services with complex dependencies
├── utils/               # Growing pile of miscellaneous functions
└── types/               # Global type soup
```

Feature-based architecture provides clear boundaries and ownership:

```
✅ Feature-Based (scales with team)
src/
├── features/
│   ├── authentication/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── utils/
│   ├── product-catalog/
│   └── shopping-cart/
├── shared/              # Only truly shared code
└── app/                 # App-level configuration
```

### Implementing Feature-Based Architecture

#### Feature Module Structure

```typescript
// Standard feature module structure
export interface FeatureModule {
  // Public API - what other features can use
  components: {
    // Only components that other features need
    ProductCard: React.ComponentType<ProductCardProps>;
    ProductSearch: React.ComponentType<ProductSearchProps>;
  };
  
  hooks: {
    // Only hooks that other features need
    useProduct: (id: string) => UseProductResult;
  };
  
  types: {
    // Only types that other features need
    Product: Product;
    ProductFilters: ProductFilters;
  };
  
  services: {
    // Only services that other features need
    productApi: ProductApiService;
  };
}

// features/product-catalog/index.ts - Feature public API
export { ProductCard, ProductGrid } from './components';
export { useProduct, useProductSearch } from './hooks';
export { productApi } from './services';
export type { Product, ProductFilters, ProductCategory } from './types';

// Internal exports (not exported from index.ts)
// - ProductCardSkeleton (internal component)
// - useProductCache (internal hook)
// - productValidation (internal utility)
```

#### Feature Boundaries and Dependencies

```typescript
// Feature dependency management
interface FeatureDependencies {
  // Explicit dependencies - better than implicit imports
  authentication: AuthenticationFeature;
  shared: SharedModule;
}

// features/shopping-cart/CartFeature.ts
export class CartFeature {
  constructor(private deps: FeatureDependencies) {}
  
  // Feature can only access explicitly injected dependencies
  async addToCart(productId: string, quantity: number) {
    // ✅ Can access auth through dependency injection
    const user = this.deps.authentication.getCurrentUser();
    
    // ✅ Can access shared utilities
    const validated = this.deps.shared.validation.validateQuantity(quantity);
    
    // ❌ Cannot directly import from other features
    // import { productApi } from '../product-catalog'; // This breaks boundaries
    
    return this.cartService.addItem(productId, validated);
  }
}
```

#### Advanced Feature Communication

```typescript
// Event-driven communication between features
interface FeatureEvent {
  type: string;
  payload: unknown;
  source: string;
  timestamp: Date;
}

class FeatureEventBus {
  private listeners = new Map<string, Set<(event: FeatureEvent) => void>>();
  
  emit(event: FeatureEvent): void {
    const listeners = this.listeners.get(event.type) || new Set();
    listeners.forEach(listener => {
      try {
        listener(event);
      } catch (error) {
        console.error('Feature event listener error:', error);
      }
    });
  }
  
  subscribe(eventType: string, listener: (event: FeatureEvent) => void): () => void {
    if (!this.listeners.has(eventType)) {
      this.listeners.set(eventType, new Set());
    }
    
    this.listeners.get(eventType)!.add(listener);
    
    // Return unsubscribe function
    return () => {
      this.listeners.get(eventType)?.delete(listener);
    };
  }
}

// Feature usage of event bus
// features/shopping-cart/hooks/useCartEvents.ts
export const useCartEvents = () => {
  const eventBus = useFeatureEventBus();
  
  const addToCart = useCallback((product: Product, quantity: number) => {
    // ... add to cart logic
    
    // Emit event for other features to react
    eventBus.emit({
      type: 'cart:item_added',
      payload: { productId: product.id, quantity, cartTotal: newTotal },
      source: 'shopping-cart',
      timestamp: new Date()
    });
  }, [eventBus]);
  
  // Listen for product updates
  useEffect(() => {
    return eventBus.subscribe('product:updated', (event) => {
      const { productId, newData } = event.payload as ProductUpdatedPayload;
      // Update cart item if it exists
      updateCartItemProduct(productId, newData);
    });
  }, [eventBus]);
  
  return { addToCart };
};
```

## 🗂️ Monorepo vs Polyrepo Strategies

### Decision Framework

| Factor | Monorepo | Polyrepo |
|--------|----------|----------|
| **Team Size** | 2-50 developers | 5+ independent teams |
| **Code Sharing** | Extensive sharing | Minimal sharing |
| **Release Coordination** | Coordinated releases | Independent releases |
| **Technology Stack** | Unified stack | Diverse stacks |
| **Development Speed** | Fast iteration | Independent pace |
| **Complexity** | High tooling complexity | Low per-repo complexity |

### Monorepo Implementation with Nx

```typescript
// nx.json - Workspace configuration
{
  "version": 2,
  "projects": {
    "web-app": "apps/web",
    "mobile-app": "apps/mobile",
    "shared-ui": "libs/shared/ui",
    "shared-utils": "libs/shared/utils",
    "product-feature": "libs/features/product",
    "cart-feature": "libs/features/cart"
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "cache": true
    },
    "test": {
      "cache": true
    },
    "lint": {
      "cache": true
    }
  }
}

// Dependency graph enforcement
// libs/features/cart/project.json
{
  "name": "cart-feature",
  "sourceRoot": "libs/features/cart/src",
  "projectType": "library",
  "tags": ["scope:cart", "type:feature"],
  "implicitDependencies": [],
  // Explicitly allowed dependencies
  "dependencies": [
    "shared-ui",
    "shared-utils",
    "shared-types"
  ]
}

// .eslintrc.json - Enforce dependency rules
{
  "rules": {
    "@nx/enforce-module-boundaries": [
      "error",
      {
        "enforceBuildableLibDependency": true,
        "allow": [],
        "depConstraints": [
          {
            "sourceTag": "type:feature",
            "onlyDependOnLibsWithTags": ["type:shared", "type:util"]
          },
          {
            "sourceTag": "scope:cart",
            "notDependOnLibsWithTags": ["scope:product", "scope:auth"]
          }
        ]
      }
    ]
  }
}
```

### Polyrepo with Shared Packages

```typescript
// Package management for polyrepo
// packages/shared-types/package.json
{
  "name": "@company/shared-types",
  "version": "1.2.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "require": "./dist/index.js",
      "import": "./dist/index.mjs"
    }
  },
  "peerDependencies": {
    "typescript": ">=4.0.0"
  }
}

// Automated package publishing
// .github/workflows/publish-shared-types.yml
name: Publish Shared Types

on:
  push:
    branches: [main]
    paths: ['packages/shared-types/**']

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          registry-url: 'https://registry.npmjs.org'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build package
        run: npm run build
      
      - name: Run tests
        run: npm test
      
      - name: Publish to NPM
        run: npm publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
      
      - name: Create GitHub Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## 🧩 Microfrontends with Module Federation

### When to Use Microfrontends

**Use microfrontends when:**
- Multiple teams need independent deployments
- Different parts of the app have different technology requirements
- You need to scale development across large teams (>50 developers)
- Different features have different release cycles

**Don't use microfrontends when:**
- Small team (< 10 developers)
- Tight coupling between features
- Shared UI components across all features
- Performance is critical (microfrontends add overhead)

### Module Federation Implementation

#### Shell Application (Host)

```typescript
// webpack.config.js - Shell app configuration
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  mode: 'development',
  devServer: {
    port: 3000,
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        'product-catalog': 'productCatalog@http://localhost:3001/remoteEntry.js',
        'shopping-cart': 'shoppingCart@http://localhost:3002/remoteEntry.js',
        'user-auth': 'userAuth@http://localhost:3003/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        '@company/design-system': { singleton: true },
        '@company/shared-types': { singleton: true },
      },
    }),
  ],
};

// src/App.tsx - Dynamic microfrontend loading
const ProductCatalog = React.lazy(() => import('product-catalog/ProductCatalog'));
const ShoppingCart = React.lazy(() => import('shopping-cart/ShoppingCart'));
const UserAuth = React.lazy(() => import('user-auth/UserAuth'));

export const App: React.FC = () => {
  return (
    <ErrorBoundary>
      <Router>
        <Header />
        <Suspense fallback={<GlobalLoadingSpinner />}>
          <Routes>
            <Route path="/products/*" element={<ProductCatalog />} />
            <Route path="/cart" element={<ShoppingCart />} />
            <Route path="/auth/*" element={<UserAuth />} />
          </Routes>
        </Suspense>
      </Router>
    </ErrorBoundary>
  );
};
```

#### Microfrontend (Remote)

```typescript
// webpack.config.js - Product catalog microfrontend
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  mode: 'development',
  devServer: {
    port: 3001,
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'productCatalog',
      filename: 'remoteEntry.js',
      exposes: {
        './ProductCatalog': './src/ProductCatalog',
        './ProductCard': './src/components/ProductCard',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
        '@company/design-system': { singleton: true },
      },
    }),
  ],
};

// src/ProductCatalog.tsx - Microfrontend entry point
export const ProductCatalog: React.FC = () => {
  const navigate = useNavigate();
  
  // Internal microfrontend routing
  return (
    <Routes>
      <Route path="/" element={<ProductGrid />} />
      <Route path="/search" element={<ProductSearch />} />
      <Route path="/:id" element={<ProductDetail />} />
    </Routes>
  );
};

// Standalone development mode
if (process.env.NODE_ENV === 'development' && !window.__POWERED_BY_QIANKUN__) {
  const root = createRoot(document.getElementById('root')!);
  root.render(
    <BrowserRouter>
      <ProductCatalog />
    </BrowserRouter>
  );
}
```

#### Cross-Microfrontend Communication

```typescript
// Shared event bus for microfrontend communication
class MicrofrontendEventBus {
  private static instance: MicrofrontendEventBus;
  private eventTarget = new EventTarget();
  
  static getInstance(): MicrofrontendEventBus {
    if (!this.instance) {
      this.instance = new MicrofrontendEventBus();
    }
    return this.instance;
  }
  
  emit<T = any>(eventType: string, data: T): void {
    const event = new CustomEvent(eventType, { detail: data });
    this.eventTarget.dispatchEvent(event);
  }
  
  subscribe<T = any>(
    eventType: string, 
    handler: (data: T) => void
  ): () => void {
    const eventHandler = (event: CustomEvent) => handler(event.detail);
    this.eventTarget.addEventListener(eventType, eventHandler);
    
    return () => {
      this.eventTarget.removeEventListener(eventType, eventHandler);
    };
  }
}

// Usage in microfrontends
// Product catalog microfrontend
export const useProductEvents = () => {
  const eventBus = MicrofrontendEventBus.getInstance();
  
  const viewProduct = useCallback((product: Product) => {
    eventBus.emit('product:viewed', {
      productId: product.id,
      timestamp: Date.now(),
      source: 'product-catalog'
    });
  }, [eventBus]);
  
  return { viewProduct };
};

// Shopping cart microfrontend
export const useCartEventHandlers = () => {
  const eventBus = MicrofrontendEventBus.getInstance();
  const [recentlyViewed, setRecentlyViewed] = useState<Product[]>([]);
  
  useEffect(() => {
    return eventBus.subscribe('product:viewed', (data: ProductViewedEvent) => {
      // Show "Recently Viewed" in cart
      setRecentlyViewed(prev => 
        [data.product, ...prev.filter(p => p.id !== data.product.id)].slice(0, 5)
      );
    });
  }, [eventBus]);
  
  return { recentlyViewed };
};
```

## 🔄 Clean Separation of Concerns

### Layered Architecture Pattern

```typescript
// 1. Presentation Layer (Components)
interface PresentationLayer {
  // Only UI logic - no business logic
  components: React.ComponentType[];
  hooks: UIHook[]; // Only UI state hooks
}

// 2. Business Logic Layer (Domain Services)
interface BusinessLogicLayer {
  services: DomainService[];
  hooks: BusinessHook[]; // Business logic hooks
  validators: Validator[];
}

// 3. Data Layer (API and State)
interface DataLayer {
  repositories: Repository[];
  adapters: ApiAdapter[];
  cache: CacheService[];
}

// Example implementation
// Presentation Layer
export const ProductCard: React.FC<ProductCardProps> = ({ productId, onAddToCart }) => {
  // ✅ Only UI state and event handlers
  const [isHovered, setIsHovered] = useState(false);
  const [imageLoaded, setImageLoaded] = useState(false);
  
  // ✅ Business logic delegated to custom hook
  const { product, addToCart, isLoading } = useProduct(productId);
  
  // ✅ UI event handlers only
  const handleAddToCart = () => {
    addToCart(1);
    onAddToCart?.(product);
  };
  
  // ✅ Pure presentation logic
  return (
    <div 
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <ProductImage 
        src={product.image}
        onLoad={() => setImageLoaded(true)}
        showPlaceholder={!imageLoaded}
      />
      <ProductInfo product={product} />
      <AddToCartButton 
        onClick={handleAddToCart}
        disabled={isLoading || !product.inStock}
        showHoverEffect={isHovered}
      />
    </div>
  );
};

// Business Logic Layer
export const useProduct = (productId: string) => {
  // ✅ Business logic and domain rules
  const productService = useProductService();
  const cartService = useCartService();
  const analytics = useAnalytics();
  
  const { data: product, isLoading, error } = useQuery(
    ['product', productId],
    () => productService.getProduct(productId),
    {
      staleTime: 5 * 60 * 1000,
      select: (data) => productService.enrichProduct(data) // Business logic
    }
  );
  
  const addToCart = useCallback(async (quantity: number) => {
    // ✅ Business validation
    if (!product || !product.inStock) {
      throw new Error('Product not available');
    }
    
    if (quantity > product.stockQuantity) {
      throw new Error('Insufficient stock');
    }
    
    // ✅ Business operation
    await cartService.addItem(productId, quantity);
    
    // ✅ Business event tracking
    analytics.track('product_added_to_cart', {
      productId,
      quantity,
      price: product.price
    });
  }, [product, productId, cartService, analytics]);
  
  return { product, addToCart, isLoading, error };
};

// Data Layer
export class ProductRepository {
  constructor(
    private apiClient: ApiClient,
    private cache: CacheService
  ) {}
  
  async getProduct(id: string): Promise<Product> {
    // ✅ Data fetching logic only
    const cacheKey = `product:${id}`;
    const cached = await this.cache.get(cacheKey);
    
    if (cached) {
      return ProductSchema.parse(cached);
    }
    
    const response = await this.apiClient.get(`/products/${id}`);
    const product = ProductSchema.parse(response.data);
    
    await this.cache.set(cacheKey, product, { ttl: 300 });
    return product;
  }
  
  async searchProducts(filters: ProductFilters): Promise<ProductSearchResult> {
    // ✅ Data transformation logic
    const queryParams = this.buildSearchQuery(filters);
    const response = await this.apiClient.get('/products/search', { params: queryParams });
    
    return ProductSearchResultSchema.parse(response.data);
  }
  
  private buildSearchQuery(filters: ProductFilters): Record<string, string> {
    // ✅ Data mapping logic
    return {
      q: filters.query || '',
      category: filters.categories?.join(',') || '',
      price_min: filters.priceRange?.min?.toString() || '',
      price_max: filters.priceRange?.max?.toString() || '',
      sort: filters.sortBy || 'relevance'
    };
  }
}
```

### Domain-Driven Design in Frontend

```typescript
// Domain entities with business logic
export class Product {
  constructor(
    private data: ProductData,
    private pricingService: PricingService
  ) {}
  
  get isOnSale(): boolean {
    return this.data.originalPrice > this.data.currentPrice;
  }
  
  get discountPercentage(): number {
    if (!this.isOnSale) return 0;
    return Math.round(((this.data.originalPrice - this.data.currentPrice) / this.data.originalPrice) * 100);
  }
  
  calculatePrice(quantity: number, userTier: CustomerTier): Money {
    // ✅ Business logic encapsulated in domain entity
    let basePrice = this.data.currentPrice * quantity;
    
    // Apply tier discounts
    const tierDiscount = this.pricingService.getTierDiscount(userTier);
    basePrice = basePrice * (1 - tierDiscount);
    
    // Apply bulk discounts
    if (quantity >= 10) {
      basePrice = basePrice * 0.95; // 5% bulk discount
    }
    
    return new Money(basePrice, this.data.currency);
  }
  
  canAddToCart(requestedQuantity: number): ValidationResult {
    // ✅ Business validation rules
    if (!this.data.isActive) {
      return ValidationResult.error('Product is no longer available');
    }
    
    if (this.data.stockQuantity < requestedQuantity) {
      return ValidationResult.error(`Only ${this.data.stockQuantity} items in stock`);
    }
    
    if (requestedQuantity > this.data.maxOrderQuantity) {
      return ValidationResult.error(`Maximum ${this.data.maxOrderQuantity} items per order`);
    }
    
    return ValidationResult.success();
  }
}

// Value objects for complex data
export class Money {
  constructor(
    private amount: number,
    private currency: Currency
  ) {
    if (amount < 0) {
      throw new Error('Money amount cannot be negative');
    }
  }
  
  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add money with different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }
  
  formatForDisplay(): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: this.currency
    }).format(this.amount);
  }
  
  equals(other: Money): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
}
```

## 🗃️ Scalable State Management

### State Management Decision Tree

```typescript
// Decision framework for state management
interface StateManagementDecision {
  // Size and complexity
  appSize: 'small' | 'medium' | 'large' | 'enterprise';
  stateComplexity: 'simple' | 'moderate' | 'complex';
  
  // Team and requirements
  teamSize: number;
  performanceRequirements: 'basic' | 'high' | 'critical';
  debuggingNeeds: 'basic' | 'advanced';
  
  // Architecture preferences
  typeScriptFirst: boolean;
  serverStateHandling: 'integrated' | 'separate';
  
  recommendation: StateManagementSolution;
}

type StateManagementSolution = 
  | 'react-state-only'
  | 'context-api'
  | 'zustand'
  | 'redux-toolkit'
  | 'valtio'
  | 'jotai';

const getStateManagementRecommendation = (criteria: Omit<StateManagementDecision, 'recommendation'>): StateManagementSolution => {
  // Small apps with simple state
  if (criteria.appSize === 'small' && criteria.stateComplexity === 'simple') {
    return 'react-state-only';
  }
  
  // Medium apps with moderate complexity
  if (criteria.appSize === 'medium' && criteria.teamSize < 10) {
    return criteria.typeScriptFirst ? 'zustand' : 'context-api';
  }
  
  // Large apps or complex state
  if (criteria.appSize === 'large' || criteria.stateComplexity === 'complex') {
    return criteria.debuggingNeeds === 'advanced' ? 'redux-toolkit' : 'zustand';
  }
  
  // Enterprise with large teams
  if (criteria.appSize === 'enterprise' && criteria.teamSize > 20) {
    return 'redux-toolkit'; // Better tooling and patterns for large teams
  }
  
  return 'zustand'; // Safe default for most cases
};
```

### Zustand Implementation for Scale

```typescript
// Modular store architecture with Zustand
interface StoreModule<T> {
  name: string;
  initialState: T;
  actions: Record<string, (...args: any[]) => void>;
}

// User store module
const createUserStoreModule = (): StoreModule<UserState> => ({
  name: 'user',
  initialState: {
    currentUser: null,
    preferences: null,
    isLoading: false,
    error: null
  },
  actions: {
    setUser: (user: User | null) => set(state => ({ ...state, currentUser: user })),
    updatePreferences: (preferences: UserPreferences) => 
      set(state => ({ ...state, preferences })),
    setLoading: (isLoading: boolean) => set(state => ({ ...state, isLoading })),
    setError: (error: string | null) => set(state => ({ ...state, error }))
  }
});

// Cart store module
const createCartStoreModule = (): StoreModule<CartState> => ({
  name: 'cart',
  initialState: {
    items: [],
    isOpen: false,
    lastUpdated: null
  },
  actions: {
    addItem: (product: Product, quantity: number) => set(state => {
      const existingItem = state.items.find(item => item.product.id === product.id);
      
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.product.id === product.id
              ? { ...item, quantity: item.quantity + quantity }
              : item
          ),
          lastUpdated: new Date()
        };
      }
      
      return {
        ...state,
        items: [...state.items, { product, quantity, addedAt: new Date() }],
        lastUpdated: new Date()
      };
    }),
    removeItem: (productId: string) => set(state => ({
      ...state,
      items: state.items.filter(item => item.product.id !== productId),
      lastUpdated: new Date()
    })),
    toggleCart: () => set(state => ({ ...state, isOpen: !state.isOpen }))
  }
});

// Combined store with persistence and devtools
export const useAppStore = create<AppState>()(
  devtools(
    persist(
      subscribeWithSelector(
        immer((set, get) => ({
          // Combine all modules
          user: createUserStoreModule().initialState,
          cart: createCartStoreModule().initialState,
          
          // Combined actions
          ...createUserStoreModule().actions,
          ...createCartStoreModule().actions,
          
          // Global actions
          reset: () => set(() => ({
            user: createUserStoreModule().initialState,
            cart: createCartStoreModule().initialState
          }))
        }))
      ),
      {
        name: 'app-store',
        version: 1,
        partialize: (state) => ({
          cart: state.cart,
          user: { preferences: state.user.preferences } // Only persist preferences
        }),
        migrate: (persistedState: any, version: number) => {
          // Handle store migrations
          if (version === 0) {
            // Migrate from version 0 to 1
            return {
              ...persistedState,
              cart: {
                ...persistedState.cart,
                lastUpdated: new Date()
              }
            };
          }
          return persistedState;
        }
      }
    ),
    {
      name: 'app-store',
      serialize: { options: true }
    }
  )
);

// Typed selectors for performance
export const selectUser = (state: AppState) => state.user;
export const selectCart = (state: AppState) => state.cart;
export const selectCartItems = (state: AppState) => state.cart.items;
export const selectCartTotal = (state: AppState) => 
  state.cart.items.reduce((total, item) => total + (item.product.price * item.quantity), 0);

// Subscription-based side effects
useAppStore.subscribe(
  (state) => state.cart.items,
  (cartItems, previousCartItems) => {
    // Auto-save cart to server when items change
    if (cartItems !== previousCartItems && cartItems.length > 0) {
      debounce(() => {
        const user = useAppStore.getState().user.currentUser;
        if (user) {
          cartApi.syncCart(user.id, cartItems);
        }
      }, 1000)();
    }
  },
  { equalityFn: shallow }
);
```

### Redux Toolkit for Enterprise Scale

```typescript
// Feature-based slice organization
// features/product-catalog/store/productSlice.ts
export const productSlice = createSlice({
  name: 'products',
  initialState: {
    entities: {} as Record<string, Product>,
    searchResults: {
      query: '',
      filters: {},
      products: [],
      totalCount: 0,
      isLoading: false
    },
    favorites: [],
    recentlyViewed: []
  } as ProductState,
  reducers: {
    // Synchronous actions
    productReceived: (state, action: PayloadAction<Product>) => {
      state.entities[action.payload.id] = action.payload;
    },
    searchStarted: (state, action: PayloadAction<SearchFilters>) => {
      state.searchResults.isLoading = true;
      state.searchResults.query = action.payload.query;
      state.searchResults.filters = action.payload;
    },
    addToFavorites: (state, action: PayloadAction<string>) => {
      if (!state.favorites.includes(action.payload)) {
        state.favorites.push(action.payload);
      }
    },
    addToRecentlyViewed: (state, action: PayloadAction<Product>) => {
      // Remove if already exists
      state.recentlyViewed = state.recentlyViewed.filter(p => p.id !== action.payload.id);
      // Add to beginning
      state.recentlyViewed.unshift(action.payload);
      // Keep only last 10
      state.recentlyViewed = state.recentlyViewed.slice(0, 10);
    }
  },
  extraReducers: (builder) => {
    // Async thunk results
    builder
      .addCase(searchProducts.pending, (state) => {
        state.searchResults.isLoading = true;
      })
      .addCase(searchProducts.fulfilled, (state, action) => {
        state.searchResults.isLoading = false;
        state.searchResults.products = action.payload.products;
        state.searchResults.totalCount = action.payload.totalCount;
        
        // Update entities
        action.payload.products.forEach(product => {
          state.entities[product.id] = product;
        });
      })
      .addCase(searchProducts.rejected, (state, action) => {
        state.searchResults.isLoading = false;
        // Handle error state
      });
  }
});

// Async thunks for side effects
export const searchProducts = createAsyncThunk(
  'products/searchProducts',
  async (filters: SearchFilters, { getState, rejectWithValue }) => {
    try {
      const response = await productApi.searchProducts(filters);
      
      // Track search analytics
      analytics.track('products_searched', {
        query: filters.query,
        resultCount: response.totalCount,
        filters: filters
      });
      
      return response;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// Memoized selectors for performance
export const selectAllProducts = (state: RootState) => 
  Object.values(state.products.entities);

export const selectProductById = createSelector(
  [(state: RootState) => state.products.entities, (_, productId: string) => productId],
  (entities, productId) => entities[productId]
);

export const selectSearchResults = (state: RootState) => 
  state.products.searchResults;

export const selectFavoriteProducts = createSelector(
  [selectAllProducts, (state: RootState) => state.products.favorites],
  (products, favoriteIds) => products.filter(product => favoriteIds.includes(product.id))
);

// Store configuration with middleware
export const store = configureStore({
  reducer: {
    products: productSlice.reducer,
    cart: cartSlice.reducer,
    user: userSlice.reducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: [FLUSH, REHYDRATE, PAUSE, PERSIST, PURGE, REGISTER],
      },
    })
    .concat(
      // Custom middleware for side effects
      createListenerMiddleware().middleware,
      // RTK Query for API state
      productApi.middleware,
      // Analytics middleware
      analyticsMiddleware
    ),
  devTools: process.env.NODE_ENV !== 'production',
});

// Listener middleware for complex side effects
const listenerMiddleware = createListenerMiddleware();

listenerMiddleware.startListening({
  actionCreator: addToCart,
  effect: async (action, listenerApi) => {
    const state = listenerApi.getState() as RootState;
    const user = state.user.currentUser;
    
    if (user) {
      // Sync cart to server
      try {
        await cartApi.addItem(user.id, action.payload.productId, action.payload.quantity);
      } catch (error) {
        // Revert optimistic update
        listenerApi.dispatch(removeFromCart(action.payload.productId));
        // Show error notification
        listenerApi.dispatch(showNotification({ 
          type: 'error', 
          message: 'Failed to add item to cart' 
        }));
      }
    }
  }
});
```

## 📝 Exercise: Architecture Assessment

Evaluate and improve the architecture of an existing project:

### Step 1: Current State Analysis
- Audit current folder structure and organization
- Identify tight coupling and unclear boundaries
- Document current state management approach
- Assess scalability bottlenecks

### Step 2: Architecture Design
- Design feature-based architecture
- Define clear module boundaries and APIs
- Choose appropriate state management solution
- Plan migration strategy

### Step 3: Implementation Plan
- Create new architecture documentation
- Implement pilot feature with new architecture
- Create migration guidelines for existing features
- Set up tooling to enforce boundaries

### Step 4: Validation
- Measure improvement in development velocity
- Assess code quality improvements
- Validate scalability improvements
- Document lessons learned

**Deliverable:** A comprehensive architecture improvement plan with before/after analysis, implementation guidelines, and measurable success criteria.

---

**Next:** [Module 12: CI/CD & DevOps](../12-cicd-devops/) - Master automated testing pipelines, deployment strategies, and DevOps practices for frontend applications.