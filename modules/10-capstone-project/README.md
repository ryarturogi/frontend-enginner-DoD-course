# Module 10: Capstone Project - Enterprise E-commerce Platform

## 🎯 Project Overview

Build a comprehensive, enterprise-grade e-commerce platform that demonstrates mastery of all course concepts. You'll create an advanced product discovery system with intelligent search, real-time inventory management, and sophisticated cart functionality that meets Fortune 500 standards.

This capstone integrates cutting-edge technologies including React 19 Server Components, Next.js 15 App Router, advanced streaming patterns, and modern observability practices to deliver a production-ready system capable of handling millions of users.

## 📋 Project Requirements

### Core Platform: Intelligent Product Discovery & Commerce Engine

Build a sophisticated e-commerce platform with:
- **AI-Powered Search**: Vector-based semantic search with natural language processing
- **Real-time Personalization**: ML-driven recommendations and dynamic pricing
- **Advanced Inventory Management**: Real-time stock tracking with predictive analytics
- **Intelligent Cart System**: Persistent cart with abandonment recovery and cross-sell optimization
- **Enterprise UX**: Streaming data, optimistic updates, and progressive web app capabilities
- **Multi-tenant Architecture**: Support for multiple storefronts and brands
- **Global Commerce**: Multi-currency, internationalization, and regional compliance

### Production Standards Required

Your implementation must demonstrate ALL course concepts:

1. **Production-Ready Mindset** (Module 1)
2. **Complete Definition of Done** (Module 2)
3. **Comprehensive Testing Strategy** (Module 3 + 4)
4. **SOLID Code Practices** (Module 5)
5. **Performance Optimization** (Module 6)
6. **Security & Accessibility** (Module 7)
7. **Complete Documentation** (Module 8)
8. **Team Workflow Ready** (Module 9)

## 🏗️ Enterprise Architecture Requirements

### Domain-Driven Design with Micro-Frontends

```
apps/
├── commerce-platform/          # Main e-commerce application
│   ├── src/
│   │   ├── app/               # Next.js 15 App Router
│   │   │   ├── (marketing)/
│   │   │   ├── (commerce)/
│   │   │   ├── (admin)/
│   │   │   └── api/
│   │   ├── domains/           # Domain-driven design
│   │   │   ├── catalog/
│   │   │   │   ├── components/
│   │   │   │   │   ├── SearchInterface/
│   │   │   │   │   ├── ProductGrid/
│   │   │   │   │   ├── FilterSystem/
│   │   │   │   │   └── RecommendationEngine/
│   │   │   │   ├── hooks/
│   │   │   │   │   ├── useSemanticSearch.ts
│   │   │   │   │   ├── useVectorSimilarity.ts
│   │   │   │   │   └── useProductAnalytics.ts
│   │   │   │   ├── services/
│   │   │   │   │   ├── catalogApi.ts
│   │   │   │   │   ├── searchEngine.ts
│   │   │   │   │   └── vectorStore.ts
│   │   │   │   ├── repositories/
│   │   │   │   │   ├── ProductRepository.ts
│   │   │   │   │   └── CategoryRepository.ts
│   │   │   │   └── types/
│   │   │   │       ├── catalog.types.ts
│   │   │   │       └── search.types.ts
│   │   │   ├── commerce/
│   │   │   │   ├── components/
│   │   │   │   │   ├── CartSystem/
│   │   │   │   │   ├── CheckoutFlow/
│   │   │   │   │   ├── PaymentGateway/
│   │   │   │   │   └── OrderManagement/
│   │   │   │   ├── hooks/
│   │   │   │   │   ├── useCartState.ts
│   │   │   │   │   ├── useCheckoutFlow.ts
│   │   │   │   │   └── usePaymentProcessing.ts
│   │   │   │   ├── services/
│   │   │   │   │   ├── cartService.ts
│   │   │   │   │   ├── paymentService.ts
│   │   │   │   │   └── orderService.ts
│   │   │   │   └── state/
│   │   │   │       ├── cartStore.ts
│   │   │   │       └── orderStore.ts
│   │   │   ├── inventory/
│   │   │   │   ├── components/
│   │   │   │   │   ├── StockTracker/
│   │   │   │   │   ├── AvailabilityIndicator/
│   │   │   │   │   └── ReorderAlerts/
│   │   │   │   ├── hooks/
│   │   │   │   │   ├── useRealTimeInventory.ts
│   │   │   │   │   ├── useStockPrediction.ts
│   │   │   │   │   └── useInventoryAlerts.ts
│   │   │   │   └── services/
│   │   │   │       ├── inventoryApi.ts
│   │   │   │       ├── warehouseSync.ts
│   │   │   │       └── stockPredictor.ts
│   │   │   ├── personalization/
│   │   │   │   ├── components/
│   │   │   │   │   ├── RecommendationWidget/
│   │   │   │   │   ├── PersonalizedPricing/
│   │   │   │   │   └── UserPreferences/
│   │   │   │   ├── hooks/
│   │   │   │   │   ├── usePersonalization.ts
│   │   │   │   │   ├── useRecommendations.ts
│   │   │   │   │   └── useBehaviorTracking.ts
│   │   │   │   └── services/
│   │   │   │       ├── mlEngine.ts
│   │   │   │       ├── personalizationApi.ts
│   │   │   │       └── behaviorAnalytics.ts
│   │   │   └── shared/
│   │   │       ├── components/
│   │   │       ├── hooks/
│   │   │       ├── utils/
│   │   │       ├── types/
│   │   │       └── constants/
│   │   ├── infrastructure/
│   │   │   ├── api/
│   │   │   │   ├── clients/
│   │   │   │   ├── middleware/
│   │   │   │   └── types/
│   │   │   ├── database/
│   │   │   │   ├── migrations/
│   │   │   │   ├── seeders/
│   │   │   │   └── schemas/
│   │   │   ├── monitoring/
│   │   │   │   ├── metrics/
│   │   │   │   ├── tracing/
│   │   │   │   └── logging/
│   │   │   └── security/
│   │   │       ├── auth/
│   │   │       ├── encryption/
│   │   │       └── validation/
│   │   └── tests/
│   │       ├── unit/
│   │       ├── integration/
│   │       ├── e2e/
│   │       └── performance/
├── micro-frontends/          # Federated modules
│   ├── product-catalog/
│   ├── user-account/
│   └── admin-dashboard/
├── shared-libraries/         # Shared packages
│   ├── ui-components/
│   ├── design-system/
│   ├── business-logic/
│   └── utilities/
└── infrastructure/          # DevOps and tooling
    ├── docker/
    ├── kubernetes/
    ├── terraform/
    └── scripts/
```

### Enterprise Technology Stack

```typescript
// Production-grade technology stack for enterprise e-commerce
interface EnterpriseTechStack {
  // Core Framework
  framework: 'Next.js 15' | 'Remix';
  language: 'TypeScript 5+';
  runtime: 'Node.js 20+' | 'Bun';
  
  // Frontend Architecture
  rendering: {
    server: 'React 19 Server Components';
    client: 'React 19 Client Components';
    streaming: 'Suspense with Server Actions';
    hydration: 'Selective hydration';
  };
  
  // Styling & Design
  styling: 'Tailwind CSS 4' | 'Stitches' | 'Vanilla Extract';
  designSystem: {
    components: 'Radix UI' | 'Headless UI';
    tokens: 'Design tokens with Theme UI';
    documentation: 'Storybook 8+';
  };
  
  // State Management
  stateManagement: {
    server: 'React Query v5' | 'SWR' | 'Apollo Client';
    client: 'Zustand' | 'Valtio' | 'Redux Toolkit';
    forms: 'React Hook Form with Zod';
    url: 'Nuqs' | 'Next.js searchParams';
  };
  
  // Data Layer
  database: {
    primary: 'PostgreSQL' | 'PlanetScale MySQL';
    cache: 'Redis' | 'Upstash';
    search: 'Elasticsearch' | 'Typesense' | 'Algolia';
    vector: 'Pinecone' | 'Weaviate' | 'Qdrant';
  };
  
  // API & Backend
  api: {
    framework: 'tRPC' | 'GraphQL with Pothos';
    validation: 'Zod schemas';
    authentication: 'NextAuth.js' | 'Clerk' | 'Auth0';
    payments: 'Stripe' | 'PayPal';
    email: 'Resend' | 'SendGrid';
  };
  
  // Testing Strategy
  testing: {
    unit: 'Vitest' | 'Jest';
    integration: 'React Testing Library';
    e2e: 'Playwright' | 'Cypress';
    visual: 'Chromatic' | 'Percy';
    performance: 'Lighthouse CI';
    load: 'k6' | 'Artillery';
  };
  
  // Quality Assurance
  quality: {
    linting: 'ESLint 9 with TypeScript';
    formatting: 'Prettier with plugins';
    typeChecking: 'TypeScript strict mode';
    security: 'Snyk' | 'OWASP ZAP';
    dependencyCheck: 'Renovate' | 'Dependabot';
  };
  
  // Performance & Monitoring
  performance: {
    bundling: 'Next.js 15 Turbopack';
    monitoring: 'Vercel Analytics' | 'Sentry Performance';
    vitals: 'Web Vitals with custom metrics';
    cdn: 'Vercel Edge' | 'Cloudflare';
    optimization: 'Sharp image processing';
  };
  
  // Security Infrastructure
  security: {
    validation: 'Zod with runtime type checking';
    sanitization: 'DOMPurify with CSP';
    authentication: 'OAuth 2.0 with PKCE';
    authorization: 'RBAC with CASL';
    encryption: 'Node.js crypto with Argon2';
    secrets: 'HashiCorp Vault' | 'AWS Secrets Manager';
    headers: 'Security headers with Helmet.js';
  };
  
  // Accessibility & Internationalization
  accessibility: {
    testing: 'axe-core with Playwright';
    implementation: 'WCAG 2.2 AA compliance';
    screenReader: 'NVDA and VoiceOver testing';
    automation: 'axe-playwright integration';
  };
  
  // DevOps & Infrastructure
  infrastructure: {
    deployment: 'Vercel' | 'Railway' | 'AWS Amplify';
    containerization: 'Docker with multi-stage builds';
    orchestration: 'Kubernetes' | 'Docker Swarm';
    cicd: 'GitHub Actions' | 'GitLab CI';
    monitoring: 'DataDog' | 'New Relic' | 'Grafana';
    logging: 'Structured logging with Pino';
  };
  
  // AI/ML Integration
  artificialIntelligence: {
    search: 'OpenAI Embeddings' | 'Cohere';
    recommendations: 'TensorFlow.js' | 'Vercel AI SDK';
    personalization: 'Custom ML models';
    analytics: 'Google Analytics 4 with ML insights';
  };
}
```

## 📝 Complete Definition of Done Checklist

### ✅ Code Quality (All Required)

- [ ] **Architecture & Organization**
  - [ ] Feature-based folder structure implemented
  - [ ] Clear separation of concerns (UI/Business Logic/Data)
  - [ ] SOLID principles applied throughout
  - [ ] Custom hooks for business logic
  - [ ] Reusable components with single responsibilities

- [ ] **TypeScript Excellence**
  - [ ] Strict mode enabled and compliant
  - [ ] Comprehensive type definitions for all data
  - [ ] Utility types used appropriately
  - [ ] No `any` types (except rare justified cases)
  - [ ] Generic types for reusability

- [ ] **Code Standards**
  - [ ] ESLint configuration with security rules
  - [ ] Prettier formatting enforced
  - [ ] Consistent naming conventions
  - [ ] JSDoc documentation for complex functions
  - [ ] No console.log or debug code

### ✅ Testing Strategy (All Required)

- [ ] **Unit Testing (>80% coverage)**
  - [ ] All custom hooks tested
  - [ ] Utility functions tested
  - [ ] API services tested
  - [ ] State management tested
  - [ ] AAA and GWT patterns used appropriately

- [ ] **Integration Testing**
  - [ ] Component interactions tested
  - [ ] User workflows tested
  - [ ] API integration tested
  - [ ] Error scenarios covered
  - [ ] Loading states tested

- [ ] **End-to-End Testing**
  - [ ] Complete search and cart flow
  - [ ] Cross-browser compatibility
  - [ ] Mobile responsiveness
  - [ ] Performance under load
  - [ ] Error recovery scenarios

- [ ] **Accessibility Testing**
  - [ ] Automated axe-core tests
  - [ ] Keyboard navigation tested
  - [ ] Screen reader compatibility
  - [ ] Color contrast verified
  - [ ] Focus management tested

### ✅ Performance Requirements (All Required)

- [ ] **Core Web Vitals Excellence**
  - [ ] LCP < 1.5s (Largest Contentful Paint)
  - [ ] INP < 100ms (Interaction to Next Paint - replacing FID)
  - [ ] CLS < 0.05 (Cumulative Layout Shift)
  - [ ] TTFB < 200ms (Time to First Byte)
  - [ ] FCP < 1.0s (First Contentful Paint)

- [ ] **Advanced Optimization Techniques**
  - [ ] React 19 Server Components for zero-bundle server logic
  - [ ] Streaming with Suspense boundaries for progressive loading
  - [ ] Edge runtime deployment for global performance
  - [ ] Bundle splitting with Next.js App Router automatic optimization
  - [ ] Image optimization with Sharp and WebP/AVIF formats
  - [ ] Critical path optimization with resource hints
  - [ ] Service Worker implementation for offline functionality

- [ ] **Real-Time Performance Monitoring**
  - [ ] Web Vitals tracking with custom metrics
  - [ ] Real User Monitoring (RUM) implementation
  - [ ] Performance regression testing in CI/CD
  - [ ] Bundle analyzer with size budgets
  - [ ] Core Web Vitals alerts and monitoring
  - [ ] Performance debugging tools integration

### ✅ Security Implementation (All Required)

- [ ] **Input Security**
  - [ ] All user inputs validated with Zod
  - [ ] XSS prevention implemented
  - [ ] SQL injection prevention (API layer)
  - [ ] CSRF protection configured

- [ ] **Data Security**
  - [ ] No sensitive data in localStorage
  - [ ] Secure token handling
  - [ ] Content Security Policy configured
  - [ ] HTTPS enforced

- [ ] **Security Testing**
  - [ ] Dependency vulnerability scanning
  - [ ] Security ESLint rules
  - [ ] Penetration testing basics
  - [ ] OWASP Top 10 compliance

### ✅ Accessibility Compliance (All Required)

- [ ] **WCAG 2.1 AA Standards**
  - [ ] Semantic HTML structure
  - [ ] Proper ARIA labels and roles
  - [ ] Keyboard navigation support
  - [ ] Screen reader compatibility
  - [ ] Color contrast ratios 4.5:1+

- [ ] **Inclusive Design**
  - [ ] Multiple ways to interact (mouse, keyboard, touch)
  - [ ] Clear error messages and instructions
  - [ ] Consistent navigation patterns
  - [ ] Support for assistive technologies

### ✅ Documentation Package (All Required)

- [ ] **Technical Documentation**
  - [ ] Comprehensive README with setup instructions
  - [ ] API documentation with examples
  - [ ] Component documentation with Storybook
  - [ ] Architecture Decision Records (ADRs)
  - [ ] Troubleshooting guide

- [ ] **Code Documentation**
  - [ ] JSDoc for all public APIs
  - [ ] Inline comments for complex logic
  - [ ] Type definitions as documentation
  - [ ] Example usage in comments

- [ ] **Process Documentation**
  - [ ] Contributing guidelines
  - [ ] Development workflow
  - [ ] Testing strategy documentation
  - [ ] Deployment procedures

### ✅ Production Readiness (All Required)

- [ ] **Monitoring & Observability**
  - [ ] Error tracking with Sentry
  - [ ] Performance monitoring
  - [ ] User analytics tracking
  - [ ] Health check endpoints

- [ ] **Deployment Ready**
  - [ ] Environment configuration
  - [ ] CI/CD pipeline setup
  - [ ] Feature flags implementation
  - [ ] Rollback procedures documented

- [ ] **Scalability Considerations**
  - [ ] Component reusability
  - [ ] State management scalability
  - [ ] API design for growth
  - [ ] Performance under load

## 🛠️ Implementation Guide

### Phase 1: Enterprise Foundation (Week 1-2)

#### Advanced Project Architecture with Domain Models

```typescript
// src/domains/shared/types/core.types.ts
export interface Product {
  id: ProductId;
  name: string;
  description: string;
  shortDescription?: string;
  price: Money;
  originalPrice?: Money;
  images: ProductImage[];
  category: ProductCategory;
  subcategories: ProductSubcategory[];
  rating: ProductRating;
  reviews: ProductReview[];
  inventory: InventoryStatus;
  variants: ProductVariant[];
  attributes: ProductAttribute[];
  seo: SEOMetadata;
  timestamps: EntityTimestamps;
  tenant: TenantId;
}

// Advanced type system with branded types
export type ProductId = string & { readonly brand: unique symbol };
export type UserId = string & { readonly brand: unique symbol };
export type TenantId = string & { readonly brand: unique symbol };

export interface Money {
  amount: number;
  currency: CurrencyCode;
  formattedValue: string;
}

export interface ProductImage {
  id: string;
  url: string;
  altText: string;
  width: number;
  height: number;
  priority: number;
  variants: ImageVariant[];
}

export interface SearchFilters {
  query: string;
  semanticQuery?: string; // AI-powered semantic search
  category?: ProductCategory[];
  priceRange?: MoneyRange;
  rating?: RatingFilter;
  availability?: AvailabilityFilter;
  attributes?: AttributeFilter[];
  sortBy: SortOption;
  pagination: PaginationParams;
  personalization?: PersonalizationContext;
}

export interface SearchResult {
  products: Product[];
  totalCount: number;
  facets: SearchFacets;
  suggestions: SearchSuggestion[];
  query: ProcessedQuery;
  filters: AppliedFilters;
  analytics: SearchAnalytics;
  recommendations: RecommendedProduct[];
}

export interface CartItem {
  id: CartItemId;
  product: Product;
  variant?: ProductVariant;
  quantity: number;
  customizations?: ProductCustomization[];
  pricing: CartItemPricing;
  addedAt: Date;
  lastUpdated: Date;
  source: CartItemSource;
}

// Vector search types for AI-powered features
export interface ProductEmbedding {
  productId: ProductId;
  vector: Float32Array;
  metadata: EmbeddingMetadata;
  lastUpdated: Date;
}

export interface SemanticSearchQuery {
  query: string;
  embedding: Float32Array;
  threshold: number;
  limit: number;
  filters?: VectorSearchFilters;
}
```

#### Next.js 15 App Router with Server Components

```typescript
// src/app/(commerce)/products/page.tsx - Server Component
import { Suspense } from 'react';
import { searchProducts } from '@/domains/catalog/services/searchService';
import { ProductGrid } from '@/domains/catalog/components/ProductGrid';
import { SearchInterface } from '@/domains/catalog/components/SearchInterface';
import { FilterSidebar } from '@/domains/catalog/components/FilterSidebar';
import { ProductGridSkeleton } from '@/domains/catalog/components/ProductGridSkeleton';

interface ProductsPageProps {
  searchParams: {
    q?: string;
    category?: string[];
    minPrice?: string;
    maxPrice?: string;
    sort?: string;
    page?: string;
  };
}

export default async function ProductsPage({ searchParams }: ProductsPageProps) {
  // Server-side search processing with caching
  const searchResult = await searchProducts({
    query: searchParams.q ?? '',
    category: searchParams.category,
    priceRange: {
      min: Number(searchParams.minPrice) || 0,
      max: Number(searchParams.maxPrice) || Infinity,
    },
    sortBy: searchParams.sort as SortOption ?? 'relevance',
    pagination: {
      page: Number(searchParams.page) || 1,
      limit: 24,
    },
  });

  return (
    <div className="flex min-h-screen">
      {/* Server Component for initial state */}
      <FilterSidebar 
        facets={searchResult.facets} 
        initialFilters={searchParams}
      />
      
      <main className="flex-1 p-6">
        {/* Client Component for interactive search */}
        <SearchInterface 
          initialQuery={searchParams.q} 
          suggestions={searchResult.suggestions}
        />
        
        {/* Suspense boundary for streaming */}
        <Suspense fallback={<ProductGridSkeleton />}>
          <ProductGrid 
            products={searchResult.products}
            totalCount={searchResult.totalCount}
            currentPage={Number(searchParams.page) || 1}
          />
        </Suspense>
      </main>
    </div>
  );
}

// src/app/(commerce)/products/loading.tsx
export default function ProductsLoading() {
  return <ProductGridSkeleton />;
}

// src/app/(commerce)/products/error.tsx
'use client';

export default function ProductsError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-[400px]">
      <h2 className="text-xl font-semibold mb-4">Something went wrong!</h2>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Try again
      </button>
    </div>
  );
}
```

#### Enterprise API Layer with tRPC

```typescript
// src/server/api/routers/catalog.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '@/server/api/trpc';
import { ProductRepository } from '@/domains/catalog/repositories/ProductRepository';
import { SearchService } from '@/domains/catalog/services/SearchService';
import { VectorSearchService } from '@/domains/catalog/services/VectorSearchService';

const searchInputSchema = z.object({
  query: z.string().min(0).max(200),
  semanticSearch: z.boolean().optional(),
  filters: z.object({
    category: z.array(z.string()).optional(),
    priceRange: z.object({
      min: z.number().min(0),
      max: z.number().min(0),
    }).optional(),
    rating: z.number().min(1).max(5).optional(),
    availability: z.enum(['in-stock', 'out-of-stock', 'backorder']).optional(),
  }).optional(),
  sort: z.enum(['relevance', 'price-asc', 'price-desc', 'rating', 'newest']).default('relevance'),
  pagination: z.object({
    page: z.number().min(1).default(1),
    limit: z.number().min(1).max(100).default(24),
  }).default({ page: 1, limit: 24 }),
});

export const catalogRouter = createTRPCRouter({
  searchProducts: publicProcedure
    .input(searchInputSchema)
    .query(async ({ input, ctx }) => {
      const { query, semanticSearch, filters, sort, pagination } = input;
      
      // Use vector search for semantic queries
      if (semanticSearch && query.trim()) {
        return await VectorSearchService.semanticSearch({
          query,
          filters,
          sort,
          pagination,
          userId: ctx.session?.user?.id,
        });
      }
      
      // Traditional full-text search
      return await SearchService.search({
        query,
        filters,
        sort,
        pagination,
        userId: ctx.session?.user?.id,
      });
    }),

  getProductById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input }) => {
      return await ProductRepository.findById(input.id);
    }),

  getRecommendations: publicProcedure
    .input(z.object({
      productId: z.string().optional(),
      userId: z.string().optional(),
      type: z.enum(['similar', 'frequently-bought', 'trending', 'personalized']),
      limit: z.number().min(1).max(20).default(8),
    }))
    .query(async ({ input }) => {
      return await RecommendationService.getRecommendations(input);
    }),
});
```

#### Enterprise Infrastructure Layer

```typescript
// src/infrastructure/api/api-client.ts - Enterprise API client with observability
import { z } from 'zod';
import { trace, context, SpanStatusCode } from '@opentelemetry/api';

export class EnterpriseApiClient {
  private baseUrl: string;
  private tracer = trace.getTracer('api-client');
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  async request<T>(
    endpoint: string, 
    options: RequestOptions = {},
    schema?: z.ZodSchema<T>
  ): Promise<T> {
    const span = this.tracer.startSpan(`api.${endpoint.replace('/', '.')}`);
    
    try {
      const startTime = performance.now();
      
      const response = await fetch(`${this.baseUrl}${endpoint}`, {
        headers: {
          'Content-Type': 'application/json',
          'X-Request-ID': crypto.randomUUID(),
          'X-Client-Version': process.env.NEXT_PUBLIC_APP_VERSION || '1.0.0',
          ...options?.headers,
        },
        ...options,
        signal: options.signal || AbortSignal.timeout(30000), // 30s timeout
      });
      
      const duration = performance.now() - startTime;
      
      // Track performance metrics
      span.setAttributes({
        'http.method': options.method || 'GET',
        'http.url': `${this.baseUrl}${endpoint}`,
        'http.status_code': response.status,
        'http.response_time_ms': duration,
      });
      
      if (!response.ok) {
        const errorData = await response.text();
        span.setStatus({ code: SpanStatusCode.ERROR, message: errorData });
        throw new ApiError(response.status, errorData, {
          endpoint,
          method: options.method || 'GET',
          duration,
        });
      }
      
      const data = await response.json();
      
      // Validate response if schema provided
      if (schema) {
        const validationResult = schema.safeParse(data);
        if (!validationResult.success) {
          span.setStatus({ 
            code: SpanStatusCode.ERROR, 
            message: 'Response validation failed' 
          });
          throw new ValidationError('Response validation failed', validationResult.error);
        }
        return validationResult.data;
      }
      
      span.setStatus({ code: SpanStatusCode.OK });
      return data;
    } catch (error) {
      span.setStatus({ 
        code: SpanStatusCode.ERROR, 
        message: error instanceof Error ? error.message : 'Unknown error' 
      });
      throw error;
    } finally {
      span.end();
    }
  }
}

// src/infrastructure/database/repositories/BaseRepository.ts
export abstract class BaseRepository<TEntity, TId> {
  protected db: Database;
  protected cache: CacheService;
  protected logger: Logger;
  
  constructor(
    db: Database,
    cache: CacheService,
    logger: Logger
  ) {
    this.db = db;
    this.cache = cache;
    this.logger = logger;
  }
  
  abstract findById(id: TId): Promise<TEntity | null>;
  abstract create(entity: Omit<TEntity, 'id'>): Promise<TEntity>;
  abstract update(id: TId, updates: Partial<TEntity>): Promise<TEntity>;
  abstract delete(id: TId): Promise<void>;
  
  protected async withCaching<T>(
    key: string,
    fetcher: () => Promise<T>,
    ttl: number = 300 // 5 minutes default
  ): Promise<T> {
    const cached = await this.cache.get<T>(key);
    if (cached) {
      this.logger.debug('Cache hit', { key });
      return cached;
    }
    
    const data = await fetcher();
    await this.cache.set(key, data, ttl);
    this.logger.debug('Cache miss, data fetched and cached', { key });
    
    return data;
  }
  
  protected async withTransaction<T>(
    operation: (trx: Transaction) => Promise<T>
  ): Promise<T> {
    return await this.db.transaction(operation);
  }
}

// src/infrastructure/monitoring/metrics.ts - Enterprise metrics collection
import { metrics } from '@opentelemetry/api';

const meter = metrics.getMeter('ecommerce-platform');

export const Metrics = {
  // Business metrics
  searchRequests: meter.createCounter('search_requests_total', {
    description: 'Total number of search requests',
  }),
  
  searchDuration: meter.createHistogram('search_duration_ms', {
    description: 'Search request duration in milliseconds',
  }),
  
  cartAdditions: meter.createCounter('cart_additions_total', {
    description: 'Total number of items added to cart',
  }),
  
  checkoutSteps: meter.createCounter('checkout_steps_total', {
    description: 'Checkout funnel steps completed',
  }),
  
  // Technical metrics
  apiRequests: meter.createCounter('api_requests_total', {
    description: 'Total API requests',
  }),
  
  cacheHitRate: meter.createGauge('cache_hit_rate', {
    description: 'Cache hit rate percentage',
  }),
  
  activeUsers: meter.createUpDownCounter('active_users', {
    description: 'Number of active users',
  }),
  
  recordSearchRequest(query: string, results: number, duration: number) {
    this.searchRequests.add(1, {
      has_query: query.length > 0 ? 'true' : 'false',
      has_results: results > 0 ? 'true' : 'false',
    });
    
    this.searchDuration.record(duration, {
      query_length: query.length > 20 ? 'long' : 'short',
    });
  },
  
  recordCartAddition(productId: string, quantity: number, source: string) {
    this.cartAdditions.add(1, {
      source,
      quantity_range: quantity > 1 ? 'multiple' : 'single',
    });
  },
};
```

### Phase 2: Core Features (Week 2-3)

#### Advanced Search Implementation

```typescript
// src/features/product-search/hooks/useProductSearch.ts
export const useProductSearch = (initialFilters: SearchFilters) => {
  const [filters, setFilters] = useState<SearchFilters>(initialFilters);
  const [searchHistory, setSearchHistory] = useLocalStorage('searchHistory', []);
  
  const { 
    data: searchResult, 
    isLoading, 
    error,
    refetch 
  } = useQuery({
    queryKey: ['products', 'search', filters],
    queryFn: () => searchProducts(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    keepPreviousData: true, // For pagination
  });
  
  const debouncedSearch = useMemo(
    () => debounce((newFilters: SearchFilters) => {
      setFilters(newFilters);
      
      // Track search analytics
      trackSearchEvent(newFilters.query, searchResult?.totalCount || 0);
      
      // Update search history
      if (newFilters.query.trim()) {
        setSearchHistory(prev => 
          [newFilters.query, ...prev.filter(q => q !== newFilters.query)].slice(0, 10)
        );
      }
    }, 300),
    [searchResult?.totalCount, setSearchHistory]
  );
  
  return {
    searchResult,
    isLoading,
    error,
    filters,
    updateFilters: debouncedSearch,
    refetch,
    searchHistory
  };
};
```

#### Shopping Cart with Persistence

```typescript
// src/features/shopping-cart/store/cartStore.ts
interface CartState {
  items: CartItem[];
  isOpen: boolean;
  lastUpdated: Date;
}

interface CartActions {
  addItem: (product: Product, quantity?: number) => void;
  removeItem: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  openCart: () => void;
  closeCart: () => void;
}

export const useCartStore = create<CartState & CartActions>()(
  persist(
    (set, get) => ({
      items: [],
      isOpen: false,
      lastUpdated: new Date(),
      
      addItem: (product, quantity = 1) => {
        const { items } = get();
        const existingItem = items.find(item => item.product.id === product.id);
        
        if (existingItem) {
          set({
            items: items.map(item =>
              item.product.id === product.id
                ? { ...item, quantity: item.quantity + quantity }
                : item
            ),
            lastUpdated: new Date()
          });
        } else {
          set({
            items: [...items, { product, quantity, addedAt: new Date() }],
            lastUpdated: new Date()
          });
        }
        
        // Analytics tracking
        trackCartEvent('add_to_cart', { 
          product_id: product.id, 
          quantity,
          cart_value: calculateCartTotal(get().items)
        });
      },
      
      // ... other cart actions
    }),
    {
      name: 'shopping-cart',
      version: 1,
      migrate: (persistedState: any, version: number) => {
        // Handle cart data migrations
        return persistedState;
      }
    }
  )
);
```

### Phase 3: Advanced Features (Week 4)

#### Performance Optimization

```typescript
// src/features/product-search/components/SearchResults/VirtualizedProductGrid.tsx
import { FixedSizeGrid as Grid } from 'react-window';

interface VirtualizedProductGridProps {
  products: Product[];
  onProductClick: (product: Product) => void;
}

export const VirtualizedProductGrid: React.FC<VirtualizedProductGridProps> = ({
  products,
  onProductClick
}) => {
  const { width, height } = useWindowSize();
  const itemsPerRow = Math.floor(width / 300); // 300px per product card
  const rowCount = Math.ceil(products.length / itemsPerRow);
  
  const ProductGridItem = memo(({ columnIndex, rowIndex, style }: GridChildComponentProps) => {
    const productIndex = rowIndex * itemsPerRow + columnIndex;
    const product = products[productIndex];
    
    if (!product) return null;
    
    return (
      <div style={style}>
        <ProductCard 
          product={product} 
          onClick={() => onProductClick(product)}
          loading="lazy"
        />
      </div>
    );
  });
  
  return (
    <Grid
      columnCount={itemsPerRow}
      columnWidth={300}
      height={height - 200} // Account for header
      rowCount={rowCount}
      rowHeight={400}
      width={width}
    >
      {ProductGridItem}
    </Grid>
  );
};
```

#### Advanced Error Handling

```typescript
// src/features/shared/components/ErrorBoundary/ErrorBoundary.tsx
export class FeatureErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.setState({ errorInfo });
    
    // Log to monitoring service
    captureException(error, {
      tags: { feature: this.props.feature },
      extra: errorInfo,
      user: getCurrentUser(),
    });
    
    // Track error analytics
    trackErrorEvent(error.message, {
      feature: this.props.feature,
      component: errorInfo.componentStack,
      userId: getCurrentUser()?.id,
    });
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback ? (
        <this.props.fallback 
          error={this.state.error!}
          retry={() => this.setState({ hasError: false, error: null })}
        />
      ) : (
        <DefaultErrorFallback 
          error={this.state.error!}
          feature={this.props.feature}
          onRetry={() => this.setState({ hasError: false, error: null })}
        />
      );
    }
    
    return this.props.children;
  }
}
```

### Phase 4: Testing & Quality (Week 5)

#### Comprehensive Testing Suite

```typescript
// src/features/product-search/components/SearchBar/SearchBar.test.tsx
describe('SearchBar Component', () => {
  const mockOnSearch = jest.fn();
  
  beforeEach(() => {
    jest.clearAllMocks();
  });
  
  describe('User Interactions', () => {
    it('should call onSearch when user types and stops typing', async () => {
      const user = userEvent.setup();
      render(<SearchBar onSearch={mockOnSearch} />);
      
      const searchInput = screen.getByRole('searchbox');
      
      await user.type(searchInput, 'laptop');
      
      // Wait for debounce
      await waitFor(() => {
        expect(mockOnSearch).toHaveBeenCalledWith('laptop');
      }, { timeout: 500 });
    });
    
    it('should show search suggestions when user types', async () => {
      const mockSuggestions = ['laptop', 'laptop bag', 'laptop stand'];
      server.use(
        rest.get('/api/search/suggestions', (req, res, ctx) => {
          return res(ctx.json({ suggestions: mockSuggestions }));
        })
      );
      
      const user = userEvent.setup();
      render(<SearchBar onSearch={mockOnSearch} />);
      
      const searchInput = screen.getByRole('searchbox');
      await user.type(searchInput, 'lap');
      
      await waitFor(() => {
        mockSuggestions.forEach(suggestion => {
          expect(screen.getByText(suggestion)).toBeInTheDocument();
        });
      });
    });
  });
  
  describe('Accessibility', () => {
    it('should be keyboard navigable', async () => {
      const user = userEvent.setup();
      render(<SearchBar onSearch={mockOnSearch} />);
      
      // Tab to search input
      await user.tab();
      expect(screen.getByRole('searchbox')).toHaveFocus();
      
      // Type to show suggestions
      await user.keyboard('laptop');
      
      // Arrow down to navigate suggestions
      await user.keyboard('{ArrowDown}');
      expect(screen.getByRole('option', { name: /laptop/i })).toHaveAttribute('aria-selected', 'true');
    });
    
    it('should announce search results to screen readers', async () => {
      render(<SearchBar onSearch={mockOnSearch} />);
      
      const resultsAnnouncement = screen.getByRole('status', { name: /search results/i });
      expect(resultsAnnouncement).toBeInTheDocument();
    });
  });
});
```

#### E2E Testing with Playwright

```typescript
// tests/e2e/product-search.spec.ts
test.describe('Product Search Flow', () => {
  test('complete search and add to cart flow', async ({ page }) => {
    await page.goto('/');
    
    // Search for products
    await page.fill('[data-testid="search-input"]', 'wireless headphones');
    await page.press('[data-testid="search-input"]', 'Enter');
    
    // Wait for search results
    await page.waitForSelector('[data-testid="search-results"]');
    
    // Verify search results are displayed
    const productCards = page.locator('[data-testid="product-card"]');
    await expect(productCards).toHaveCount.greaterThan(0);
    
    // Apply filters
    await page.click('[data-testid="filter-category-electronics"]');
    await page.fill('[data-testid="filter-price-min"]', '50');
    await page.fill('[data-testid="filter-price-max"]', '200');
    
    // Wait for filtered results
    await page.waitForResponse(response => 
      response.url().includes('/api/search') && response.status() === 200
    );
    
    // Add first product to cart
    await productCards.first().click();
    await page.waitForSelector('[data-testid="product-detail"]');
    await page.click('[data-testid="add-to-cart-button"]');
    
    // Verify cart is updated
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');
    
    // Open cart and verify item
    await page.click('[data-testid="cart-icon"]');
    await expect(page.locator('[data-testid="cart-item"]')).toBeVisible();
  });
  
  test('should handle search errors gracefully', async ({ page }) => {
    // Mock API to return error
    await page.route('/api/search**', route => {
      route.fulfill({
        status: 500,
        body: JSON.stringify({ error: 'Internal server error' })
      });
    });
    
    await page.goto('/');
    await page.fill('[data-testid="search-input"]', 'test query');
    await page.press('[data-testid="search-input"]', 'Enter');
    
    // Verify error message is shown
    await expect(page.locator('[data-testid="error-message"]')).toBeVisible();
    await expect(page.locator('[data-testid="error-message"]')).toContainText('search failed');
  });
});
```

### Phase 5: Documentation & Deployment (Week 6)

#### Complete Documentation Package

```markdown
# Advanced Product Search & Cart - Technical Documentation

## Architecture Overview

This feature implements a production-ready product search system with advanced filtering and integrated shopping cart functionality. The implementation follows enterprise-level patterns and practices.

### Key Design Decisions

#### ADR-001: Feature-Based Architecture
**Status**: Accepted
**Context**: Need scalable architecture for growing e-commerce platform
**Decision**: Implement feature-based folder structure with clear boundaries
**Consequences**: Better maintainability, easier testing, clearer ownership

#### ADR-002: Client-Side Search State Management
**Status**: Accepted  
**Context**: Need responsive search experience with complex filters
**Decision**: Use React Query for server state, Zustand for UI state
**Consequences**: Optimistic updates, better caching, complex state coordination

### Performance Characteristics

- **Search Response Time**: < 300ms (p95)
- **Bundle Size Impact**: +45KB gzipped
- **Core Web Vitals**: LCP < 2.5s, FID < 100ms, CLS < 0.1
- **Memory Usage**: < 10MB additional heap

### Security Considerations

- All search queries sanitized to prevent XSS
- Rate limiting implemented (100 requests/minute)
- No sensitive data cached in localStorage
- CSP headers prevent unauthorized script execution

### API Documentation

#### Search Endpoint
```typescript
GET /api/products/search?q={query}&category[]={cat}&price_min={min}&price_max={max}

Response:
{
  "products": Product[],
  "totalCount": number,
  "facets": SearchFacets,
  "suggestions": string[]
}
```
```

#### CI/CD Pipeline Setup

```yaml
# .github/workflows/capstone-ci.yml
name: Capstone Project CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  quality-gates:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js 18
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Code Quality Checks
        run: |
          pnpm run lint
          pnpm run type-check
          pnpm run format:check
      
      - name: Security Audit
        run: pnpm audit --audit-level moderate
      
      - name: Unit & Integration Tests
        run: pnpm run test:coverage
      
      - name: Build Application
        run: pnpm run build
      
      - name: Bundle Size Check
        run: pnpm run analyze:bundle
      
      - name: E2E Tests
        run: |
          pnpm run start &
          sleep 30
          pnpm run test:e2e:ci
      
      - name: Accessibility Tests
        run: pnpm run test:a11y
      
      - name: Performance Tests
        run: pnpm run lighthouse:ci
      
      - name: Validate DoD Compliance
        run: node scripts/validate-capstone-dod.js
```

## 🎯 Deliverables Checklist

### Code Deliverables
- [ ] Complete feature implementation with all requirements
- [ ] Comprehensive test suite (unit, integration, e2e, a11y)
- [ ] Performance optimizations and monitoring
- [ ] Security implementations and validations
- [ ] Complete TypeScript coverage with strict mode

### Documentation Deliverables
- [ ] Project README with setup and architecture
- [ ] API documentation with examples
- [ ] Component documentation (Storybook recommended)
- [ ] Architecture Decision Records (ADRs)
- [ ] Testing strategy documentation
- [ ] Performance analysis report
- [ ] Security review documentation
- [ ] Accessibility compliance report

### Process Deliverables
- [ ] CI/CD pipeline with all quality gates
- [ ] Feature flag implementation
- [ ] Monitoring and error tracking setup
- [ ] Deployment documentation
- [ ] Rollback procedures
- [ ] Team handoff documentation

### Presentation Deliverables
- [ ] Demo video showing complete functionality
- [ ] Code walkthrough presentation
- [ ] Architecture explanation
- [ ] Performance metrics presentation
- [ ] DoD compliance demonstration

## 🚀 Bonus Challenges

### Advanced Features (Optional)
- [ ] **Search Analytics Dashboard**: Real-time search metrics and insights
- [ ] **Personalized Recommendations**: ML-based product suggestions
- [ ] **Voice Search**: Speech-to-text search capability
- [ ] **Progressive Web App**: Offline search and cart persistence
- [ ] **Internationalization**: Multi-language search support

### Technical Excellence (Optional)
- [ ] **Micro-frontend Architecture**: Module federation implementation
- [ ] **Advanced Caching**: Service worker with intelligent caching
- [ ] **Real-time Updates**: WebSocket integration for live inventory
- [ ] **A/B Testing Framework**: Feature flag-based experimentation
- [ ] **Advanced Monitoring**: Custom metrics and alerting

## 📊 Enterprise Success Criteria

### Technical Excellence Metrics
- **Test Coverage**: >95% overall, >98% for critical business paths
- **Performance**: All Core Web Vitals green with enterprise thresholds
  - LCP < 1.5s, INP < 100ms, CLS < 0.05, TTFB < 200ms
- **Accessibility**: WCAG 2.2 AA compliant with automated and manual validation
- **Security**: Zero high/critical vulnerabilities, SOC 2 Type II compliance
- **Code Quality**: ESLint score >98%, TypeScript strict mode, 0 code smells
- **Bundle Optimization**: <400KB initial bundle, <50KB per route

### Business Performance Metrics
- **Search Success Rate**: >90% of searches return relevant results
- **Semantic Search Accuracy**: >85% relevance score for AI-powered queries
- **Cart Conversion**: >70% of cart additions lead to checkout intent
- **Personalization Effectiveness**: >25% lift in conversion through ML recommendations
- **User Experience**: <1.5s time to interactive, <0.05 CLS, >98% availability
- **Error Rate**: <0.05% unhandled errors, <100ms error recovery time
- **Real User Monitoring**: 75th percentile performance meets targets globally

### Enterprise Process Metrics  
- **Definition of Done Compliance**: 100% for all required criteria
- **Documentation Coverage**: All public APIs documented with examples
- **CI/CD Success**: >99.5% pipeline success rate with automated rollbacks
- **Code Review**: 100% of changes reviewed with quality gates
- **Security Scanning**: Automated scanning on every commit with blocking gates
- **Deployment Frequency**: Multiple deployments per day with zero downtime
- **Mean Time to Recovery**: <30 minutes for critical issues

### Platform Scalability Metrics
- **Concurrent Users**: Support 10,000+ concurrent users
- **Search Throughput**: >1000 queries per second with <200ms latency
- **Database Performance**: <50ms query response time at 95th percentile
- **Cache Hit Rate**: >90% for frequently accessed data
- **API Rate Limiting**: Graceful degradation under high load
- **Global Performance**: <2s load time from any geographic location

## 🎓 Enterprise-Grade Final Evaluation

Your capstone project will be evaluated against Fortune 500 enterprise standards:

### 1. **Technical Excellence & Architecture** (35%)
   - **Domain-Driven Design**: Clear domain boundaries, proper abstraction layers
   - **Scalable Architecture**: Micro-frontend readiness, modular design patterns
   - **Performance Engineering**: Sub-second load times, optimized bundle sizes
   - **Security Implementation**: Zero-trust principles, comprehensive threat modeling
   - **Accessibility Leadership**: WCAG 2.2 AA+ compliance with inclusive design

### 2. **Quality Engineering & Testing** (25%)
   - **Comprehensive Test Strategy**: >95% coverage with meaningful test scenarios
   - **Advanced Testing Patterns**: Contract testing, visual regression, performance testing
   - **Quality Gates**: Automated quality assurance with blocking CI/CD gates
   - **Test Automation**: Full E2E automation with cross-browser compatibility
   - **Chaos Engineering**: Fault tolerance and resilience testing

### 3. **Production Readiness & Operations** (25%)
   - **Enterprise CI/CD**: Blue-green deployments, automated rollbacks, feature flags
   - **Observability Excellence**: Distributed tracing, custom metrics, alerting
   - **Documentation Standards**: Comprehensive API docs, ADRs, runbooks
   - **Security Operations**: Vulnerability scanning, dependency management, compliance
   - **Performance Monitoring**: Real user monitoring, synthetic testing, SLA tracking

### 4. **Innovation & Modern Practices** (15%)
   - **AI/ML Integration**: Semantic search, personalization, intelligent features
   - **Modern Framework Usage**: React 19 Server Components, Next.js 15 App Router
   - **Developer Experience**: Advanced tooling, automation, quality-of-life improvements
   - **Business Impact**: Measurable improvements in user experience and conversion
   - **Technical Leadership**: Knowledge sharing, mentoring capability, thought leadership

### Evaluation Criteria

#### **Exceeds Expectations (90-100%)**
- Implements cutting-edge enterprise patterns that could be adopted industry-wide
- Demonstrates deep understanding of scalability, security, and performance at scale
- Creates reusable components and patterns that benefit the entire organization
- Shows innovation in problem-solving with measurable business impact
- Documentation and presentation quality suitable for executive stakeholders

#### **Meets Expectations (75-89%)**
- Successfully implements all required enterprise-grade features
- Demonstrates solid understanding of production readiness concepts
- Code quality and architecture meet professional standards
- Testing strategy covers critical paths with appropriate automation
- Documentation enables effective team handoff and maintenance

#### **Below Expectations (<75%)**
- Missing critical enterprise requirements or significant quality gaps
- Architecture or implementation patterns that wouldn't scale in production
- Insufficient testing coverage or poor test quality
- Security vulnerabilities or accessibility compliance failures
- Documentation gaps that would block production deployment

### **Portfolio Impact**

This capstone project serves as your **flagship portfolio piece** demonstrating:

✅ **Enterprise-Grade Development Skills**  
✅ **Production-Ready Code Quality**  
✅ **Modern Technology Stack Mastery**  
✅ **Security & Accessibility Leadership**  
✅ **Performance Engineering Expertise**  
✅ **DevOps & Operational Excellence**  
✅ **Business-Focused Technical Decision Making**

**Ready to build enterprise software that powers millions of users? Start your capstone project and establish yourself as an elite frontend engineer capable of delivering Fortune 500-quality solutions!**

---

**🚀 Success in this capstone project proves you can architect, build, and deploy production-ready e-commerce platforms that meet the highest enterprise standards and deliver exceptional business value.**