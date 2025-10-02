# Module 11: Enterprise Architecture & Scalable Systems

## 🎯 Learning Objectives

By the end of this module, you will:
- Master domain-driven design principles for frontend applications at enterprise scale
- Architect resilient systems using hexagonal architecture and clean architecture patterns
- Implement advanced micro-frontend architectures with Module Federation and runtime orchestration
- Design fault-tolerant, distributed frontend systems with graceful degradation
- Build scalable event-driven architectures with CQRS and event sourcing patterns
- Create self-healing systems with circuit breakers, retry mechanisms, and chaos engineering
- Implement comprehensive observability and telemetry for distributed frontend systems
- Design architecture that scales from thousands to millions of concurrent users

## 🏛️ Enterprise Architecture Principles

### Architectural Thinking at Scale

Enterprise architecture goes beyond code organization—it's about building systems that can evolve, scale, and adapt to changing business requirements while maintaining stability and performance under extreme load.

#### Core Architectural Principles

```typescript
// Enterprise Architecture Principles
interface EnterpriseArchitecture {
  // Scalability Principles
  scalability: {
    horizontal: 'Scale by adding more instances';
    vertical: 'Scale by adding more power to existing instances';
    elastic: 'Auto-scale based on demand';
    distributed: 'Distribute load across multiple systems';
  };
  
  // Reliability Principles
  reliability: {
    faultTolerance: 'System continues operating despite component failures';
    gracefulDegradation: 'Reduced functionality rather than complete failure';
    circuitBreakers: 'Prevent cascading failures';
    bulkheading: 'Isolate failures to prevent system-wide impact';
  };
  
  // Performance Principles
  performance: {
    caching: 'Multiple layers of intelligent caching';
    cdnOptimization: 'Global content distribution';
    bundleOptimization: 'Optimal code splitting and loading strategies';
    resourceHints: 'Predictive resource loading';
  };
  
  // Security Principles
  security: {
    zeroTrust: 'Never trust, always verify';
    depthInDefense: 'Multiple layers of security controls';
    principleOfLeastPrivilege: 'Minimum necessary access';
    securityByDesign: 'Security built into architecture, not bolted on';
  };
  
  // Observability Principles
  observability: {
    telemetry: 'Comprehensive metrics, logs, and traces';
    monitoring: 'Real-time system health visibility';
    alerting: 'Proactive issue detection and notification';
    debugging: 'Rapid issue identification and resolution';
  };
}
```

### Hexagonal Architecture for Frontend

```typescript
// Hexagonal Architecture (Ports and Adapters) for Frontend Applications
// Core Domain - Business Logic (Center)
export interface ProductDomainService {
  searchProducts(criteria: SearchCriteria): Promise<SearchResult>;
  getProductDetails(id: ProductId): Promise<Product>;
  addToCart(productId: ProductId, quantity: number): Promise<CartResult>;
}

// Ports - Interfaces (Define contracts)
export interface ProductRepository {
  findById(id: ProductId): Promise<Product | null>;
  search(criteria: SearchCriteria): Promise<Product[]>;
  save(product: Product): Promise<void>;
}

export interface CartRepository {
  getCart(userId: UserId): Promise<Cart>;
  addItem(userId: UserId, item: CartItem): Promise<void>;
  removeItem(userId: UserId, itemId: string): Promise<void>;
}

export interface NotificationPort {
  sendNotification(message: NotificationMessage): Promise<void>;
}

export interface AnalyticsPort {
  trackEvent(event: AnalyticsEvent): void;
  trackUserBehavior(behavior: UserBehavior): void;
}

// Adapters - Implementations (Outer layer)
export class HttpProductRepository implements ProductRepository {
  constructor(private httpClient: HttpClient) {}
  
  async findById(id: ProductId): Promise<Product | null> {
    try {
      const response = await this.httpClient.get(`/api/products/${id}`);
      return ProductMapper.toDomain(response.data);
    } catch (error) {
      if (error.status === 404) return null;
      throw new ProductRepositoryError('Failed to fetch product', error);
    }
  }
  
  async search(criteria: SearchCriteria): Promise<Product[]> {
    const queryParams = SearchCriteriaMapper.toQueryParams(criteria);
    const response = await this.httpClient.get('/api/products/search', queryParams);
    return response.data.map(ProductMapper.toDomain);
  }
}

export class IndexedDBCartRepository implements CartRepository {
  private dbName = 'ecommerce-cart';
  private version = 1;
  
  async getCart(userId: UserId): Promise<Cart> {
    const db = await this.openDatabase();
    const transaction = db.transaction(['carts'], 'readonly');
    const store = transaction.objectStore('carts');
    const cartData = await store.get(userId);
    
    return cartData ? CartMapper.toDomain(cartData) : new Cart(userId);
  }
  
  async addItem(userId: UserId, item: CartItem): Promise<void> {
    const cart = await this.getCart(userId);
    cart.addItem(item);
    
    const db = await this.openDatabase();
    const transaction = db.transaction(['carts'], 'readwrite');
    const store = transaction.objectStore('carts');
    await store.put(CartMapper.toPersistence(cart));
  }
}

// Domain Service Implementation
export class ProductDomainServiceImpl implements ProductDomainService {
  constructor(
    private productRepo: ProductRepository,
    private cartRepo: CartRepository,
    private notifications: NotificationPort,
    private analytics: AnalyticsPort
  ) {}
  
  async searchProducts(criteria: SearchCriteria): Promise<SearchResult> {
    // Domain validation
    if (!criteria.isValid()) {
      throw new InvalidSearchCriteriaError(criteria.getValidationErrors());
    }
    
    // Track search analytics
    this.analytics.trackEvent({
      type: 'product_search',
      data: { query: criteria.query, filters: criteria.filters }
    });
    
    // Execute search
    const products = await this.productRepo.search(criteria);
    
    // Apply business rules
    const filteredProducts = products.filter(product => 
      product.isAvailable() && product.meetsQualityStandards()
    );
    
    return new SearchResult(filteredProducts, criteria);
  }
  
  async addToCart(productId: ProductId, quantity: number): Promise<CartResult> {
    // Fetch product with business rules validation
    const product = await this.productRepo.findById(productId);
    if (!product) {
      throw new ProductNotFoundError(productId);
    }
    
    // Domain validation
    const validationResult = product.canAddToCart(quantity);
    if (!validationResult.isValid) {
      throw new InvalidCartOperationError(validationResult.errors);
    }
    
    // Get current user (from context or auth service)
    const userId = this.getCurrentUserId();
    const cartItem = new CartItem(product, quantity);
    
    // Add to cart
    await this.cartRepo.addItem(userId, cartItem);
    
    // Business events
    this.analytics.trackEvent({
      type: 'product_added_to_cart',
      data: { productId, quantity, userId }
    });
    
    // Send notification for high-value items
    if (product.price.isHighValue()) {
      await this.notifications.sendNotification({
        type: 'high_value_cart_addition',
        userId,
        data: { product, quantity }
      });
    }
    
    return new CartResult(true, 'Product added to cart successfully');
  }
}
```

### Clean Architecture Implementation

```typescript
// Clean Architecture Layers for Frontend

// 1. Entities (Core Business Objects)
export class Product {
  constructor(
    private id: ProductId,
    private name: string,
    private price: Money,
    private inventory: Inventory,
    private specifications: ProductSpecifications
  ) {}
  
  // Business logic methods
  isAvailable(): boolean {
    return this.inventory.hasStock() && !this.isDiscontinued();
  }
  
  canAddToCart(quantity: number): ValidationResult {
    if (quantity <= 0) {
      return ValidationResult.failure('Quantity must be positive');
    }
    
    if (!this.isAvailable()) {
      return ValidationResult.failure('Product is not available');
    }
    
    if (quantity > this.inventory.availableQuantity) {
      return ValidationResult.failure(`Only ${this.inventory.availableQuantity} items available`);
    }
    
    return ValidationResult.success();
  }
  
  calculateTotalPrice(quantity: number, discounts: Discount[]): Money {
    let basePrice = this.price.multiply(quantity);
    
    for (const discount of discounts) {
      if (discount.appliesTo(this)) {
        basePrice = discount.apply(basePrice);
      }
    }
    
    return basePrice;
  }
}

// 2. Use Cases (Application Business Rules)
export class SearchProductsUseCase {
  constructor(
    private productRepository: ProductRepository,
    private searchAnalytics: SearchAnalyticsService,
    private cacheService: CacheService,
    private logger: Logger
  ) {}
  
  async execute(request: SearchProductsRequest): Promise<SearchProductsResponse> {
    const startTime = performance.now();
    
    try {
      // Validate input
      const validationResult = this.validateRequest(request);
      if (!validationResult.isValid) {
        throw new ValidationError(validationResult.errors);
      }
      
      // Check cache first
      const cacheKey = this.generateCacheKey(request);
      const cachedResult = await this.cacheService.get<SearchResult>(cacheKey);
      
      if (cachedResult && !this.isCacheStale(cachedResult)) {
        this.logger.info('Cache hit for search request', { cacheKey });
        return this.mapToResponse(cachedResult);
      }
      
      // Execute search
      const searchCriteria = this.mapToSearchCriteria(request);
      const products = await this.productRepository.search(searchCriteria);
      
      // Apply business filters
      const filteredProducts = this.applyBusinessFilters(products, request);
      
      // Sort results
      const sortedProducts = this.applySorting(filteredProducts, request.sortBy);
      
      // Paginate
      const paginatedResult = this.applyPagination(sortedProducts, request.pagination);
      
      // Cache result
      const searchResult = new SearchResult(paginatedResult, request);
      await this.cacheService.set(cacheKey, searchResult, { ttl: 300 }); // 5 minutes
      
      // Track analytics
      this.searchAnalytics.trackSearch({
        query: request.query,
        filters: request.filters,
        resultCount: paginatedResult.length,
        duration: performance.now() - startTime
      });
      
      return this.mapToResponse(searchResult);
      
    } catch (error) {
      this.logger.error('Search products failed', { request, error });
      throw error;
    }
  }
  
  private applyBusinessFilters(products: Product[], request: SearchProductsRequest): Product[] {
    return products.filter(product => {
      // Apply availability filter
      if (request.onlyAvailable && !product.isAvailable()) {
        return false;
      }
      
      // Apply quality standards
      if (!product.meetsQualityStandards()) {
        return false;
      }
      
      // Apply regional restrictions
      if (!product.isAvailableInRegion(request.region)) {
        return false;
      }
      
      return true;
    });
  }
}

// 3. Interface Adapters (Adapters/Controllers)
export class ProductSearchController {
  constructor(private searchProductsUseCase: SearchProductsUseCase) {}
  
  async searchProducts(httpRequest: HttpRequest): Promise<HttpResponse> {
    try {
      // Map HTTP request to use case request
      const searchRequest = this.mapHttpRequestToUseCaseRequest(httpRequest);
      
      // Execute use case
      const response = await this.searchProductsUseCase.execute(searchRequest);
      
      // Map use case response to HTTP response
      return {
        status: 200,
        data: this.mapUseCaseResponseToHttpResponse(response),
        headers: {
          'Cache-Control': 'public, max-age=300',
          'X-Total-Count': response.totalCount.toString()
        }
      };
      
    } catch (error) {
      return this.handleError(error);
    }
  }
  
  private handleError(error: Error): HttpResponse {
    if (error instanceof ValidationError) {
      return {
        status: 400,
        data: { error: 'Invalid request', details: error.details }
      };
    }
    
    if (error instanceof ProductNotFoundError) {
      return {
        status: 404,
        data: { error: 'Product not found' }
      };
    }
    
    // Log unexpected errors
    console.error('Unexpected error in product search', error);
    
    return {
      status: 500,
      data: { error: 'Internal server error' }
    };
  }
}

// 4. Frameworks & Drivers (External interfaces)
export class NextJsProductSearchAdapter {
  constructor(private controller: ProductSearchController) {}
  
  // Next.js API route handler
  async handle(req: NextApiRequest, res: NextApiResponse) {
    const httpRequest = {
      query: req.query,
      body: req.body,
      headers: req.headers,
      method: req.method
    };
    
    const httpResponse = await this.controller.searchProducts(httpRequest);
    
    res.status(httpResponse.status);
    
    if (httpResponse.headers) {
      Object.entries(httpResponse.headers).forEach(([key, value]) => {
        res.setHeader(key, value);
      });
    }
    
    res.json(httpResponse.data);
  }
}
```

## 🛡️ Fault-Tolerant & Resilient Systems

### Circuit Breaker Pattern Implementation

```typescript
// Circuit Breaker for Frontend Services
export class CircuitBreaker {
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  private failureCount = 0;
  private lastFailureTime?: Date;
  private nextAttemptTime?: Date;
  
  constructor(
    private failureThreshold: number = 5,
    private timeout: number = 60000, // 1 minute
    private monitoringWindow: number = 120000 // 2 minutes
  ) {}
  
  async execute<T>(operation: () => Promise<T>, fallback?: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (this.shouldAttemptReset()) {
        this.state = 'HALF_OPEN';
      } else {
        return this.handleOpenCircuit(fallback);
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      
      if (fallback) {
        try {
          return await fallback();
        } catch (fallbackError) {
          throw error; // Throw original error if fallback fails
        }
      }
      
      throw error;
    }
  }
  
  private onSuccess(): void {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  private onFailure(): void {
    this.failureCount++;
    this.lastFailureTime = new Date();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
      this.nextAttemptTime = new Date(Date.now() + this.timeout);
    }
  }
  
  private shouldAttemptReset(): boolean {
    return this.nextAttemptTime ? Date.now() >= this.nextAttemptTime.getTime() : false;
  }
  
  private async handleOpenCircuit<T>(fallback?: () => Promise<T>): Promise<T> {
    if (fallback) {
      return await fallback();
    }
    throw new CircuitBreakerOpenError('Circuit breaker is OPEN. Service temporarily unavailable.');
  }
}

// Service implementation with circuit breaker
export class ResilientProductService {
  private searchCircuitBreaker = new CircuitBreaker(3, 30000); // 3 failures, 30s timeout
  private detailsCircuitBreaker = new CircuitBreaker(5, 60000); // 5 failures, 1min timeout
  
  constructor(
    private productApi: ProductApiService,
    private cacheService: CacheService,
    private fallbackService: FallbackProductService
  ) {}
  
  async searchProducts(criteria: SearchCriteria): Promise<SearchResult> {
    return this.searchCircuitBreaker.execute(
      async () => {
        const result = await this.productApi.search(criteria);
        
        // Cache successful results
        await this.cacheService.set(
          `search:${this.hashCriteria(criteria)}`,
          result,
          { ttl: 300 }
        );
        
        return result;
      },
      async () => {
        // Fallback to cache or degraded service
        const cachedResult = await this.cacheService.get(`search:${this.hashCriteria(criteria)}`);
        
        if (cachedResult) {
          return { ...cachedResult, fromCache: true };
        }
        
        // Fallback to simplified search
        return this.fallbackService.basicSearch(criteria);
      }
    );
  }
  
  async getProductDetails(productId: string): Promise<Product> {
    return this.detailsCircuitBreaker.execute(
      async () => {
        return await this.productApi.getProductDetails(productId);
      },
      async () => {
        // Fallback to cached data or basic info
        const cached = await this.cacheService.get(`product:${productId}`);
        if (cached) {
          return { ...cached, degraded: true };
        }
        
        return this.fallbackService.getBasicProductInfo(productId);
      }
    );
  }
}
```

### Retry Mechanisms with Exponential Backoff

```typescript
// Advanced Retry Strategy
export class RetryStrategy {
  constructor(
    private maxRetries: number = 3,
    private baseDelay: number = 1000,
    private maxDelay: number = 30000,
    private backoffMultiplier: number = 2,
    private jitterFactor: number = 0.1
  ) {}
  
  async execute<T>(
    operation: () => Promise<T>,
    shouldRetry: (error: Error, attempt: number) => boolean = this.defaultShouldRetry
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 1; attempt <= this.maxRetries + 1; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error as Error;
        
        if (attempt > this.maxRetries || !shouldRetry(error as Error, attempt)) {
          throw error;
        }
        
        const delay = this.calculateDelay(attempt);
        await this.sleep(delay);
      }
    }
    
    throw lastError!;
  }
  
  private calculateDelay(attempt: number): number {
    // Exponential backoff with jitter
    let delay = Math.min(
      this.baseDelay * Math.pow(this.backoffMultiplier, attempt - 1),
      this.maxDelay
    );
    
    // Add jitter to prevent thundering herd
    const jitter = delay * this.jitterFactor * Math.random();
    delay += jitter;
    
    return Math.floor(delay);
  }
  
  private defaultShouldRetry(error: Error, attempt: number): boolean {
    // Retry on network errors, 5xx errors, but not on 4xx client errors
    if (error.name === 'NetworkError' || error.name === 'TimeoutError') {
      return true;
    }
    
    if ('status' in error) {
      const status = (error as any).status;
      return status >= 500 || status === 408 || status === 429; // Server errors, timeout, rate limit
    }
    
    return false;
  }
  
  private sleep(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Implementation with monitoring
export class ResilientApiClient {
  private retryStrategy = new RetryStrategy(3, 1000, 10000);
  
  constructor(
    private httpClient: HttpClient,
    private metrics: MetricsService,
    private logger: Logger
  ) {}
  
  async request<T>(
    endpoint: string,
    options: RequestOptions = {}
  ): Promise<T> {
    const startTime = performance.now();
    const requestId = crypto.randomUUID();
    
    this.logger.debug('API request started', { endpoint, requestId });
    
    try {
      const result = await this.retryStrategy.execute(
        async () => {
          const response = await this.httpClient.request(endpoint, {
            ...options,
            timeout: options.timeout || 10000,
            headers: {
              ...options.headers,
              'X-Request-ID': requestId,
            }
          });
          
          return response;
        },
        (error, attempt) => {
          this.logger.warn('API request failed, considering retry', {
            endpoint,
            requestId,
            attempt,
            error: error.message
          });
          
          // Custom retry logic
          if (endpoint.includes('/critical/')) {
            return attempt <= 5; // More retries for critical endpoints
          }
          
          return this.defaultRetryLogic(error, attempt);
        }
      );
      
      const duration = performance.now() - startTime;
      
      this.metrics.recordApiRequest({
        endpoint,
        status: 'success',
        duration,
        requestId
      });
      
      this.logger.debug('API request succeeded', { endpoint, requestId, duration });
      
      return result;
      
    } catch (error) {
      const duration = performance.now() - startTime;
      
      this.metrics.recordApiRequest({
        endpoint,
        status: 'failure',
        duration,
        error: error.message,
        requestId
      });
      
      this.logger.error('API request failed after retries', {
        endpoint,
        requestId,
        duration,
        error
      });
      
      throw error;
    }
  }
  
  private defaultRetryLogic(error: Error, attempt: number): boolean {
    // Don't retry auth errors or validation errors
    if ('status' in error) {
      const status = (error as any).status;
      if (status === 401 || status === 403 || (status >= 400 && status < 500)) {
        return false;
      }
    }
    
    return attempt <= 3;
  }
}
```

### Graceful Degradation Strategies

```typescript
// Feature Flag System for Graceful Degradation
export class FeatureFlagService {
  private flags = new Map<string, FeatureFlag>();
  private cache = new Map<string, { value: boolean; expires: number }>();
  
  constructor(
    private flagProvider: FeatureFlagProvider,
    private logger: Logger
  ) {}
  
  async isEnabled(flagName: string, context?: FeatureFlagContext): Promise<boolean> {
    try {
      // Check cache first
      const cached = this.cache.get(flagName);
      if (cached && cached.expires > Date.now()) {
        return cached.value;
      }
      
      // Fetch flag value
      const flag = await this.flagProvider.getFlag(flagName);
      const enabled = this.evaluateFlag(flag, context);
      
      // Cache result
      this.cache.set(flagName, {
        value: enabled,
        expires: Date.now() + 30000 // 30 seconds
      });
      
      return enabled;
      
    } catch (error) {
      this.logger.error('Feature flag evaluation failed', { flagName, error });
      
      // Fail open or closed based on flag configuration
      const fallback = this.getFallbackValue(flagName);
      return fallback;
    }
  }
  
  private evaluateFlag(flag: FeatureFlag, context?: FeatureFlagContext): boolean {
    if (!flag.enabled) return false;
    
    // Percentage rollout
    if (flag.rolloutPercentage < 100) {
      const hash = this.hashContext(context);
      const bucket = hash % 100;
      if (bucket >= flag.rolloutPercentage) return false;
    }
    
    // Target conditions
    if (flag.conditions && context) {
      return this.evaluateConditions(flag.conditions, context);
    }
    
    return true;
  }
  
  private getFallbackValue(flagName: string): boolean {
    // Define safe fallbacks for critical features
    const safeFallbacks = {
      'ai-search': false, // Disable AI search if flag service fails
      'advanced-filters': false, // Disable advanced filtering
      'real-time-inventory': false, // Fall back to cached inventory
      'personalization': false, // Disable personalization
      'payment-processing': true, // Keep payment enabled (critical)
      'user-authentication': true, // Keep auth enabled (critical)
    };
    
    return safeFallbacks[flagName] ?? false;
  }
}

// Degraded Service Implementation
export class DegradedProductService {
  constructor(
    private featureFlags: FeatureFlagService,
    private primaryService: ProductService,
    private fallbackService: BasicProductService,
    private cacheService: CacheService
  ) {}
  
  async searchProducts(criteria: SearchCriteria): Promise<SearchResult> {
    // Check if AI search is available
    const aiSearchEnabled = await this.featureFlags.isEnabled('ai-search', {
      userId: criteria.userId,
      region: criteria.region
    });
    
    if (aiSearchEnabled) {
      try {
        return await this.primaryService.searchWithAI(criteria);
      } catch (error) {
        // Log degradation
        console.warn('AI search failed, falling back to basic search', error);
      }
    }
    
    // Fallback to basic search
    return this.fallbackService.basicSearch(criteria);
  }
  
  async getProductRecommendations(productId: string): Promise<Product[]> {
    const personalizationEnabled = await this.featureFlags.isEnabled('personalization');
    
    if (personalizationEnabled) {
      try {
        return await this.primaryService.getPersonalizedRecommendations(productId);
      } catch (error) {
        console.warn('Personalization failed, using static recommendations', error);
      }
    }
    
    // Fallback to static popular products
    return this.fallbackService.getPopularProducts();
  }
  
  async getInventoryStatus(productId: string): Promise<InventoryStatus> {
    const realTimeEnabled = await this.featureFlags.isEnabled('real-time-inventory');
    
    if (realTimeEnabled) {
      try {
        return await this.primaryService.getRealTimeInventory(productId);
      } catch (error) {
        console.warn('Real-time inventory failed, using cached data', error);
      }
    }
    
    // Fallback to cached inventory data
    const cached = await this.cacheService.get(`inventory:${productId}`);
    return cached || { status: 'unknown', quantity: 0, lastUpdated: new Date() };
  }
}
```

### Event-Driven Architecture with CQRS

```typescript
// Command Query Responsibility Segregation (CQRS) Implementation
// Commands - Write operations
export interface AddToCartCommand {
  userId: string;
  productId: string;
  quantity: number;
  sessionId: string;
  timestamp: Date;
}

export interface UpdateInventoryCommand {
  productId: string;
  newQuantity: number;
  reason: 'sale' | 'restock' | 'adjustment';
  operatorId: string;
}

// Events - Domain events
export interface ProductAddedToCartEvent {
  eventId: string;
  eventType: 'ProductAddedToCart';
  aggregateId: string; // cartId
  aggregateVersion: number;
  data: {
    userId: string;
    productId: string;
    quantity: number;
    price: number;
    timestamp: Date;
  };
  metadata: {
    correlationId: string;
    causationId: string;
    userId: string;
  };
}

export interface InventoryUpdatedEvent {
  eventId: string;
  eventType: 'InventoryUpdated';
  aggregateId: string; // productId
  aggregateVersion: number;
  data: {
    productId: string;
    previousQuantity: number;
    newQuantity: number;
    reason: string;
    timestamp: Date;
  };
}

// Event Store
export class EventStore {
  private events: Map<string, Event[]> = new Map();
  private eventBus: EventBus;
  
  constructor(eventBus: EventBus) {
    this.eventBus = eventBus;
  }
  
  async appendEvents(aggregateId: string, events: Event[], expectedVersion: number): Promise<void> {
    const existingEvents = this.events.get(aggregateId) || [];
    
    if (existingEvents.length !== expectedVersion) {
      throw new ConcurrencyError('Aggregate version mismatch');
    }
    
    // Append events
    const updatedEvents = [...existingEvents, ...events];
    this.events.set(aggregateId, updatedEvents);
    
    // Publish events to event bus
    for (const event of events) {
      await this.eventBus.publish(event);
    }
  }
  
  async getEvents(aggregateId: string, fromVersion: number = 0): Promise<Event[]> {
    const events = this.events.get(aggregateId) || [];
    return events.slice(fromVersion);
  }
  
  async getSnapshot(aggregateId: string): Promise<AggregateSnapshot | null> {
    // Implementation for snapshot loading
    return null;
  }
}

// Aggregate Root
export class CartAggregate {
  private id: string;
  private version: number = 0;
  private items: CartItem[] = [];
  private uncommittedEvents: Event[] = [];
  
  constructor(id: string) {
    this.id = id;
  }
  
  static fromHistory(events: Event[]): CartAggregate {
    const cart = new CartAggregate(events[0]?.aggregateId || '');
    
    for (const event of events) {
      cart.applyEvent(event);
      cart.version++;
    }
    
    return cart;
  }
  
  addProduct(productId: string, quantity: number, userId: string): void {
    // Business logic validation
    if (quantity <= 0) {
      throw new InvalidOperationError('Quantity must be positive');
    }
    
    const existingItem = this.items.find(item => item.productId === productId);
    
    if (existingItem) {
      // Update quantity
      const newQuantity = existingItem.quantity + quantity;
      
      if (newQuantity > 10) { // Business rule: max 10 items per product
        throw new BusinessRuleViolationError('Maximum 10 items per product allowed');
      }
      
      this.raiseEvent({
        eventId: crypto.randomUUID(),
        eventType: 'CartItemQuantityUpdated',
        aggregateId: this.id,
        aggregateVersion: this.version + 1,
        data: {
          productId,
          previousQuantity: existingItem.quantity,
          newQuantity,
          userId,
          timestamp: new Date()
        }
      });
    } else {
      // Add new item
      this.raiseEvent({
        eventId: crypto.randomUUID(),
        eventType: 'ProductAddedToCart',
        aggregateId: this.id,
        aggregateVersion: this.version + 1,
        data: {
          productId,
          quantity,
          userId,
          timestamp: new Date()
        }
      });
    }
  }
  
  private raiseEvent(event: Event): void {
    this.uncommittedEvents.push(event);
    this.applyEvent(event);
  }
  
  private applyEvent(event: Event): void {
    switch (event.eventType) {
      case 'ProductAddedToCart':
        this.items.push({
          productId: event.data.productId,
          quantity: event.data.quantity,
          addedAt: event.data.timestamp
        });
        break;
        
      case 'CartItemQuantityUpdated':
        const item = this.items.find(i => i.productId === event.data.productId);
        if (item) {
          item.quantity = event.data.newQuantity;
        }
        break;
    }
  }
  
  getUncommittedEvents(): Event[] {
    return this.uncommittedEvents;
  }
  
  markEventsAsCommitted(): void {
    this.uncommittedEvents = [];
  }
}

// Command Handler
export class AddToCartCommandHandler {
  constructor(
    private eventStore: EventStore,
    private productService: ProductService
  ) {}
  
  async handle(command: AddToCartCommand): Promise<void> {
    // Load aggregate
    const events = await this.eventStore.getEvents(command.userId);
    const cart = CartAggregate.fromHistory(events);
    
    // Validate product exists and is available
    const product = await this.productService.getProduct(command.productId);
    if (!product || !product.isAvailable()) {
      throw new ProductNotAvailableError(command.productId);
    }
    
    // Execute business logic
    cart.addProduct(command.productId, command.quantity, command.userId);
    
    // Persist events
    const uncommittedEvents = cart.getUncommittedEvents();
    await this.eventStore.appendEvents(command.userId, uncommittedEvents, cart.version);
    
    cart.markEventsAsCommitted();
  }
}

// Query Side - Read Models
export class CartReadModel {
  constructor(private database: Database) {}
  
  async getCart(userId: string): Promise<CartView | null> {
    const result = await this.database.query(
      'SELECT * FROM cart_views WHERE user_id = ?',
      [userId]
    );
    
    return result[0] || null;
  }
  
  async getCartItems(userId: string): Promise<CartItemView[]> {
    return await this.database.query(
      'SELECT * FROM cart_item_views WHERE user_id = ? ORDER BY added_at DESC',
      [userId]
    );
  }
}

// Event Projection
export class CartProjection {
  constructor(private database: Database) {}
  
  async project(event: Event): Promise<void> {
    switch (event.eventType) {
      case 'ProductAddedToCart':
        await this.handleProductAddedToCart(event as ProductAddedToCartEvent);
        break;
        
      case 'CartItemQuantityUpdated':
        await this.handleCartItemQuantityUpdated(event);
        break;
    }
  }
  
  private async handleProductAddedToCart(event: ProductAddedToCartEvent): Promise<void> {
    await this.database.execute(
      `INSERT INTO cart_item_views (user_id, product_id, quantity, added_at, last_updated)
       VALUES (?, ?, ?, ?, ?)
       ON DUPLICATE KEY UPDATE quantity = quantity + VALUES(quantity), last_updated = VALUES(last_updated)`,
      [
        event.data.userId,
        event.data.productId,
        event.data.quantity,
        event.data.timestamp,
        event.data.timestamp
      ]
    );
    
    // Update cart totals
    await this.updateCartTotals(event.data.userId);
  }
  
  private async updateCartTotals(userId: string): Promise<void> {
    // Calculate and update cart totals in read model
    const totals = await this.database.query(
      `SELECT COUNT(*) as item_count, SUM(quantity) as total_quantity, SUM(price * quantity) as total_amount
       FROM cart_item_views WHERE user_id = ?`,
      [userId]
    );
    
    await this.database.execute(
      `INSERT INTO cart_views (user_id, item_count, total_quantity, total_amount, last_updated)
       VALUES (?, ?, ?, ?, NOW())
       ON DUPLICATE KEY UPDATE 
         item_count = VALUES(item_count),
         total_quantity = VALUES(total_quantity),
         total_amount = VALUES(total_amount),
         last_updated = VALUES(last_updated)`,
      [userId, totals[0].item_count, totals[0].total_quantity, totals[0].total_amount]
    );
  }
}
```

## 🏗️ Advanced Feature-Based Architecture

### Domain-Driven Design for Frontend

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