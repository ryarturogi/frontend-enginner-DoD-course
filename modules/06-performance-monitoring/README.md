# Module 6: Performance & Monitoring

## 🎯 Learning Objectives

By the end of this module, you will:
- Master Core Web Vitals and understand their business impact
- Implement performance budgets and monitoring systems
- Apply advanced caching strategies for optimal performance
- Set up comprehensive error monitoring and observability
- Optimize loading performance with modern techniques
- Monitor and improve real user experience metrics

## 📊 Core Web Vitals Deep Dive

Core Web Vitals are the essential metrics that Google uses to measure user experience and search ranking.

### Largest Contentful Paint (LCP) - Loading Performance
**Target: < 2.5 seconds**

LCP measures when the largest content element becomes visible to users.

#### Common LCP Elements
- Large images or image carousels
- Large text blocks or headers
- Video thumbnail images
- Background images loaded via CSS

#### LCP Optimization Strategies

```typescript
// 1. Image Optimization
const OptimizedImage: React.FC<{
  src: string;
  alt: string;
  priority?: boolean;
  sizes?: string;
}> = ({ src, alt, priority = false, sizes }) => {
  return (
    <picture>
      {/* Modern formats for browsers that support them */}
      <source 
        srcSet={`${src}?format=avif&w=400 400w, ${src}?format=avif&w=800 800w`}
        type="image/avif"
        sizes={sizes}
      />
      <source 
        srcSet={`${src}?format=webp&w=400 400w, ${src}?format=webp&w=800 800w`}
        type="image/webp"
        sizes={sizes}
      />
      <img
        src={`${src}?w=800`}
        alt={alt}
        loading={priority ? 'eager' : 'lazy'}
        decoding="async"
        style={{ contentVisibility: priority ? 'visible' : 'auto' }}
      />
    </picture>
  );
};

// 2. Critical Resource Preloading
const usePreloadCriticalResources = () => {
  useEffect(() => {
    // Preload hero image
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = 'image';
    link.href = '/images/hero-banner.webp';
    document.head.appendChild(link);
    
    // Preload critical fonts
    const fontLink = document.createElement('link');
    fontLink.rel = 'preload';
    fontLink.as = 'font';
    fontLink.type = 'font/woff2';
    fontLink.href = '/fonts/inter-var.woff2';
    fontLink.crossOrigin = 'anonymous';
    document.head.appendChild(fontLink);
    
    return () => {
      document.head.removeChild(link);
      document.head.removeChild(fontLink);
    };
  }, []);
};

// 3. Server-Side Rendering Optimization
const ProductPage: React.FC<{ productId: string }> = ({ productId }) => {
  // Critical data fetched on server
  const { data: product } = useSWR(
    `/api/products/${productId}`,
    fetcher,
    {
      fallbackData: ssrProduct, // Provided by SSR
      revalidateOnMount: false   // Don't refetch immediately
    }
  );
  
  return (
    <div>
      {/* Above-the-fold content rendered on server */}
      <ProductHero product={product} />
      
      {/* Below-the-fold content loaded after */}
      <Suspense fallback={<ProductDetailsSkeleton />}>
        <ProductDetails productId={productId} />
      </Suspense>
    </div>
  );
};
```

### First Input Delay (FID) - Interactivity
**Target: < 100 milliseconds**

FID measures the delay between user interaction and browser response.

#### FID Optimization Strategies

```typescript
// 1. Code Splitting and Lazy Loading
const LazyProductReviews = lazy(() => 
  import('./ProductReviews').then(module => ({
    default: module.ProductReviews
  }))
);

const LazyRecommendations = lazy(() => 
  import('./Recommendations').then(module => ({
    default: module.Recommendations
  }))
);

// 2. Optimize JavaScript Execution
const useOptimizedEventHandler = () => {
  // Debounce expensive operations
  const debouncedSearch = useMemo(
    () => debounce((query: string) => {
      // Expensive search operation
      performSearch(query);
    }, 300),
    []
  );
  
  // Use requestIdleCallback for non-critical work
  const performBackgroundTask = useCallback((task: () => void) => {
    if ('requestIdleCallback' in window) {
      requestIdleCallback(task, { timeout: 5000 });
    } else {
      // Fallback for browsers without requestIdleCallback
      setTimeout(task, 0);
    }
  }, []);
  
  return { debouncedSearch, performBackgroundTask };
};

// 3. Web Workers for Heavy Computations
const useWebWorkerCalculation = () => {
  const [result, setResult] = useState<number | null>(null);
  const [isCalculating, setIsCalculating] = useState(false);
  
  const calculate = useCallback((data: number[]) => {
    setIsCalculating(true);
    
    const worker = new Worker('/workers/calculation-worker.js');
    
    worker.postMessage(data);
    
    worker.onmessage = (e) => {
      setResult(e.data);
      setIsCalculating(false);
      worker.terminate();
    };
    
    worker.onerror = () => {
      setIsCalculating(false);
      worker.terminate();
    };
  }, []);
  
  return { result, isCalculating, calculate };
};

// Web Worker file: /public/workers/calculation-worker.js
self.onmessage = function(e) {
  const data = e.data;
  
  // Heavy calculation that would block main thread
  let result = 0;
  for (let i = 0; i < data.length; i++) {
    result += Math.sqrt(data[i]) * Math.random();
  }
  
  self.postMessage(result);
};
```

### Cumulative Layout Shift (CLS) - Visual Stability
**Target: < 0.1**

CLS measures unexpected layout shifts during page loading.

#### CLS Optimization Strategies

```typescript
// 1. Skeleton Loading with Exact Dimensions
const ProductCardSkeleton: React.FC = () => (
  <div 
    className="product-card-skeleton"
    style={{
      width: '300px',
      height: '400px', // Exact dimensions to prevent shifts
      display: 'flex',
      flexDirection: 'column'
    }}
  >
    <div 
      className="skeleton-image"
      style={{ width: '100%', height: '200px', backgroundColor: '#f0f0f0' }}
    />
    <div 
      className="skeleton-title"
      style={{ width: '80%', height: '20px', backgroundColor: '#f0f0f0', margin: '10px 0' }}
    />
    <div 
      className="skeleton-price"
      style={{ width: '60%', height: '16px', backgroundColor: '#f0f0f0' }}
    />
  </div>
);

// 2. Aspect Ratio Containers
const AspectRatioImage: React.FC<{
  src: string;
  alt: string;
  aspectRatio: string; // e.g., "16/9", "4/3"
}> = ({ src, alt, aspectRatio }) => (
  <div 
    style={{ 
      aspectRatio,
      position: 'relative',
      overflow: 'hidden'
    }}
  >
    <img
      src={src}
      alt={alt}
      style={{
        position: 'absolute',
        top: 0,
        left: 0,
        width: '100%',
        height: '100%',
        objectFit: 'cover'
      }}
      onLoad={(e) => {
        // Image loaded, no layout shift
        e.currentTarget.style.opacity = '1';
      }}
      style={{
        opacity: 0,
        transition: 'opacity 0.3s ease-in-out'
      }}
    />
  </div>
);

// 3. Font Loading Optimization
const FontLoader: React.FC = () => {
  useEffect(() => {
    // Load fonts with font-display: swap
    const fontFace = new FontFace(
      'Inter',
      'url(/fonts/inter-var.woff2) format("woff2")',
      {
        display: 'swap',
        unicodeRange: 'U+0000-00FF'
      }
    );
    
    fontFace.load().then(() => {
      document.fonts.add(fontFace);
      document.body.classList.add('fonts-loaded');
    });
  }, []);
  
  return null;
};

// CSS for preventing layout shift during font loading
/* 
.fonts-loaded {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
}
*/
```

### Interaction to Next Paint (INP) - Responsiveness
**Target: < 200 milliseconds**

INP measures the latency of all interactions throughout the page lifecycle.

```typescript
// 1. Optimize Event Handlers
const useOptimizedClick = (handler: () => void) => {
  return useCallback((e: React.MouseEvent) => {
    e.preventDefault();
    
    // Use scheduler to break up work
    if ('scheduler' in window && 'postTask' in (window as any).scheduler) {
      (window as any).scheduler.postTask(handler, { priority: 'user-blocking' });
    } else {
      // Fallback: use setTimeout to yield to browser
      setTimeout(handler, 0);
    }
  }, [handler]);
};

// 2. Virtualized Lists for Large Data
const VirtualizedProductList: React.FC<{ products: Product[] }> = ({ products }) => {
  const [startIndex, setStartIndex] = useState(0);
  const [endIndex, setEndIndex] = useState(10);
  const containerRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    const container = containerRef.current;
    if (!container) return;
    
    const handleScroll = throttle(() => {
      const scrollTop = container.scrollTop;
      const itemHeight = 100; // Height of each product card
      const containerHeight = container.clientHeight;
      
      const newStartIndex = Math.floor(scrollTop / itemHeight);
      const newEndIndex = Math.min(
        newStartIndex + Math.ceil(containerHeight / itemHeight) + 2,
        products.length
      );
      
      setStartIndex(newStartIndex);
      setEndIndex(newEndIndex);
    }, 16); // ~60fps
    
    container.addEventListener('scroll', handleScroll);
    return () => container.removeEventListener('scroll', handleScroll);
  }, [products.length]);
  
  const visibleProducts = products.slice(startIndex, endIndex);
  
  return (
    <div 
      ref={containerRef}
      style={{ 
        height: '400px', 
        overflowY: 'auto',
        position: 'relative'
      }}
    >
      <div style={{ height: startIndex * 100 }} /> {/* Spacer */}
      {visibleProducts.map((product, index) => (
        <ProductCard 
          key={product.id} 
          product={product}
          style={{ height: '100px' }}
        />
      ))}
      <div style={{ height: (products.length - endIndex) * 100 }} /> {/* Spacer */}
    </div>
  );
};
```

## 💰 Performance Budgets

Performance budgets set limits on metrics to prevent performance regression.

### Setting Up Performance Budgets

```typescript
// performance-budget.config.js
export const performanceBudgets = {
  // Bundle size budgets (in KB)
  bundles: {
    main: 250,
    vendor: 300,
    css: 50,
    images: 500
  },
  
  // Runtime performance budgets
  metrics: {
    LCP: 2500,        // milliseconds
    FID: 100,         // milliseconds
    CLS: 0.1,         // layout shift score
    INP: 200,         // milliseconds
    TTFB: 600,        // milliseconds
    FCP: 1500         // milliseconds
  },
  
  // Resource budgets
  resources: {
    totalRequests: 50,
    totalSize: 1000,  // KB
    scripts: 400,     // KB
    stylesheets: 100, // KB
    fonts: 100,       // KB
    images: 400       // KB
  }
};

// Webpack bundle analyzer integration
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: process.env.ANALYZE ? 'server' : 'disabled',
      generateStatsFile: true,
      statsOptions: { source: false }
    })
  ],
  
  performance: {
    maxAssetSize: 250000,      // 250KB
    maxEntrypointSize: 250000, // 250KB
    hints: process.env.NODE_ENV === 'production' ? 'error' : 'warning'
  }
};
```

### Lighthouse CI Integration

```yaml
# .github/workflows/performance.yml
name: Performance Monitoring

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lighthouse:
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
      
      - name: Build application
        run: pnpm build
      
      - name: Start server
        run: pnpm start &
        
      - name: Wait for server
        run: npx wait-on http://localhost:3000
      
      - name: Run Lighthouse CI
        run: npx lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: [
        'http://localhost:3000',
        'http://localhost:3000/products',
        'http://localhost:3000/checkout'
      ],
      numberOfRuns: 3
    },
    assert: {
      assertions: {
        'categories:performance': ['error', { minScore: 0.9 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.9 }],
        
        // Core Web Vitals
        'largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'first-input-delay': ['error', { maxNumericValue: 100 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        
        // Resource budgets
        'total-byte-weight': ['error', { maxNumericValue: 1000000 }], // 1MB
        'dom-size': ['error', { maxNumericValue: 1500 }],
        'script-treemap-data.unusedBytes': ['error', { maxNumericValue: 50000 }]
      }
    },
    upload: {
      target: 'temporary-public-storage'
    }
  }
};
```

## 🗄️ Advanced Caching Strategies

### HTTP Caching Headers

```typescript
// Next.js API route with caching
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const productId = searchParams.get('id');
  
  const product = await getProduct(productId);
  
  return new Response(JSON.stringify(product), {
    headers: {
      'Content-Type': 'application/json',
      // Cache for 1 hour, but allow stale content for 24 hours while revalidating
      'Cache-Control': 'public, max-age=3600, stale-while-revalidate=86400',
      // ETags for conditional requests
      'ETag': `"${product.updatedAt}"`,
      // Last modified for conditional requests
      'Last-Modified': new Date(product.updatedAt).toUTCString()
    }
  });
}

// Service Worker caching strategy
// public/sw.js
const CACHE_NAME = 'app-cache-v1';
const STATIC_CACHE = 'static-cache-v1';
const DYNAMIC_CACHE = 'dynamic-cache-v1';

// Install event - cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(STATIC_CACHE).then((cache) => {
      return cache.addAll([
        '/',
        '/static/css/main.css',
        '/static/js/main.js',
        '/images/logo.svg',
        '/manifest.json'
      ]);
    })
  );
});

// Fetch event - implement caching strategies
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Static assets - cache first
  if (request.destination === 'script' || 
      request.destination === 'style' ||
      request.destination === 'image') {
    event.respondWith(
      caches.match(request).then((response) => {
        return response || fetch(request).then((fetchResponse) => {
          const responseClone = fetchResponse.clone();
          caches.open(STATIC_CACHE).then((cache) => {
            cache.put(request, responseClone);
          });
          return fetchResponse;
        });
      })
    );
    return;
  }
  
  // API requests - network first with cache fallback
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(
      fetch(request)
        .then((response) => {
          // Only cache successful responses
          if (response.status === 200) {
            const responseClone = response.clone();
            caches.open(DYNAMIC_CACHE).then((cache) => {
              cache.put(request, responseClone);
            });
          }
          return response;
        })
        .catch(() => {
          // Network failed, try cache
          return caches.match(request);
        })
    );
    return;
  }
  
  // HTML pages - stale while revalidate
  if (request.destination === 'document') {
    event.respondWith(
      caches.open(CACHE_NAME).then((cache) => {
        return cache.match(request).then((cachedResponse) => {
          const fetchPromise = fetch(request).then((networkResponse) => {
            cache.put(request, networkResponse.clone());
            return networkResponse;
          });
          
          // Return cached version immediately if available
          return cachedResponse || fetchPromise;
        });
      })
    );
  }
});
```

### Client-Side Caching with SWR

```typescript
// Advanced SWR configuration
import useSWR, { SWRConfiguration, mutate } from 'swr';

const defaultSWRConfig: SWRConfiguration = {
  // Revalidation settings
  revalidateOnFocus: false,
  revalidateOnReconnect: true,
  revalidateIfStale: true,
  
  // Cache settings
  dedupingInterval: 2000,
  focusThrottleInterval: 5000,
  
  // Error retry
  errorRetryCount: 3,
  errorRetryInterval: 1000,
  
  // Background revalidation
  refreshInterval: 0, // Disable automatic refresh by default
  
  // Global error handler
  onError: (error, key) => {
    console.error(`SWR error for key ${key}:`, error);
    // Send to monitoring service
    if (process.env.NODE_ENV === 'production') {
      reportError(error, { context: 'SWR', key });
    }
  }
};

// Custom hooks with caching strategies
export const useProduct = (productId: string) => {
  return useSWR(
    productId ? `/api/products/${productId}` : null,
    fetcher,
    {
      ...defaultSWRConfig,
      // Cache product data for 10 minutes
      dedupingInterval: 10 * 60 * 1000,
      // Revalidate every 30 minutes
      refreshInterval: 30 * 60 * 1000,
      // Keep previous data while loading new data
      keepPreviousData: true
    }
  );
};

export const useProductList = (filters: ProductFilters) => {
  const cacheKey = useMemo(() => 
    `/api/products?${new URLSearchParams(filters).toString()}`,
    [filters]
  );
  
  return useSWR(cacheKey, fetcher, {
    ...defaultSWRConfig,
    // Shorter cache for dynamic lists
    dedupingInterval: 5 * 60 * 1000,
    // Revalidate on filter changes
    revalidateOnMount: true
  });
};

// Optimistic updates
export const useProductMutation = () => {
  const updateProduct = async (productId: string, updates: Partial<Product>) => {
    const cacheKey = `/api/products/${productId}`;
    
    // Get current data
    const currentProduct = mutate(cacheKey);
    
    // Optimistic update
    mutate(cacheKey, { ...currentProduct, ...updates }, false);
    
    try {
      // Make API call
      const updatedProduct = await fetch(`/api/products/${productId}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates)
      }).then(res => res.json());
      
      // Update cache with server response
      mutate(cacheKey, updatedProduct, false);
      
      return updatedProduct;
    } catch (error) {
      // Revert optimistic update on error
      mutate(cacheKey, currentProduct, false);
      throw error;
    }
  };
  
  return { updateProduct };
};
```

## 📊 Error Monitoring and Observability

### Comprehensive Error Tracking

```typescript
// Error monitoring setup
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

// Initialize Sentry
Sentry.init({
  dsn: process.env.REACT_APP_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.REACT_APP_VERSION,
  
  integrations: [
    new BrowserTracing({
      // Capture navigation and page load performance
      routingInstrumentation: Sentry.reactRouterV6Instrumentation(
        React.useEffect,
        useLocation,
        useNavigationType,
        createRoutesFromChildren,
        matchRoutes
      ),
      
      // Capture user interactions
      tracingOrigins: ['localhost', 'api.myapp.com'],
      
      // Capture Core Web Vitals
      _experiments: {
        enableLongTask: true,
      },
    })
  ],
  
  // Performance monitoring
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  
  // Release health monitoring
  autoSessionTracking: true,
  
  // Custom error filtering
  beforeSend(event, hint) {
    // Filter out development errors
    if (process.env.NODE_ENV === 'development') {
      console.error('Sentry Event:', event, hint);
      return null;
    }
    
    // Filter out known non-critical errors
    const ignoredErrors = [
      'ResizeObserver loop limit exceeded',
      'Non-Error promise rejection captured',
      'ChunkLoadError'
    ];
    
    if (ignoredErrors.some(error => 
      event.exception?.values?.[0]?.value?.includes(error)
    )) {
      return null;
    }
    
    return event;
  },
  
  // User context
  initialScope: {
    tags: {
      component: 'frontend'
    }
  }
});

// Custom error boundary with Sentry integration
const SentryErrorBoundary = Sentry.withErrorBoundary(App, {
  fallback: ({ error, resetError }) => (
    <div className="error-boundary">
      <h1>Something went wrong</h1>
      <p>{error.message}</p>
      <button onClick={resetError}>Try again</button>
    </div>
  ),
  beforeCapture: (scope, error, errorInfo) => {
    scope.setTag('errorBoundary', true);
    scope.setLevel('error');
    scope.setContext('errorInfo', errorInfo);
  }
});

// Performance monitoring hook
export const usePerformanceMonitoring = () => {
  useEffect(() => {
    // Monitor Core Web Vitals
    if ('web-vitals' in window) {
      import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
        getCLS((metric) => {
          Sentry.addBreadcrumb({
            category: 'web-vitals',
            message: `CLS: ${metric.value}`,
            level: 'info',
            data: metric
          });
        });
        
        getFID((metric) => {
          Sentry.addBreadcrumb({
            category: 'web-vitals',
            message: `FID: ${metric.value}ms`,
            level: 'info',
            data: metric
          });
        });
        
        getLCP((metric) => {
          Sentry.addBreadcrumb({
            category: 'web-vitals',
            message: `LCP: ${metric.value}ms`,
            level: 'info',
            data: metric
          });
        });
      });
    }
    
    // Monitor resource loading errors
    const handleResourceError = (event: ErrorEvent) => {
      Sentry.captureException(new Error(`Resource failed to load: ${event.filename}`), {
        tags: { type: 'resource-error' },
        extra: { filename: event.filename, lineno: event.lineno }
      });
    };
    
    window.addEventListener('error', handleResourceError);
    
    return () => {
      window.removeEventListener('error', handleResourceError);
    };
  }, []);
};

// Custom metrics tracking
export const trackCustomMetric = (name: string, value: number, tags?: Record<string, string>) => {
  // Send to Sentry
  Sentry.addBreadcrumb({
    category: 'custom-metric',
    message: `${name}: ${value}`,
    level: 'info',
    data: { name, value, tags }
  });
  
  // Send to analytics service
  if (typeof gtag !== 'undefined') {
    gtag('event', 'custom_metric', {
      metric_name: name,
      metric_value: value,
      ...tags
    });
  }
  
  // Send to custom analytics endpoint
  fetch('/api/metrics', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name,
      value,
      tags,
      timestamp: Date.now(),
      userId: getCurrentUserId(),
      sessionId: getSessionId()
    })
  }).catch(console.error);
};
```

### Real User Monitoring (RUM)

```typescript
// RUM implementation
class RealUserMonitoring {
  private sessionId: string;
  private userId: string | null;
  private metrics: Map<string, number> = new Map();
  
  constructor() {
    this.sessionId = this.generateSessionId();
    this.userId = this.getCurrentUserId();
    this.initializeMonitoring();
  }
  
  private initializeMonitoring() {
    // Navigation timing
    this.trackNavigationTiming();
    
    // Resource timing
    this.trackResourceTiming();
    
    // User interactions
    this.trackUserInteractions();
    
    // Long tasks
    this.trackLongTasks();
    
    // Memory usage
    this.trackMemoryUsage();
  }
  
  private trackNavigationTiming() {
    window.addEventListener('load', () => {
      const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
      
      this.recordMetric('ttfb', navigation.responseStart - navigation.requestStart);
      this.recordMetric('domContentLoaded', navigation.domContentLoadedEventEnd - navigation.navigationStart);
      this.recordMetric('loadComplete', navigation.loadEventEnd - navigation.navigationStart);
      this.recordMetric('domInteractive', navigation.domInteractive - navigation.navigationStart);
    });
  }
  
  private trackResourceTiming() {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        if (entry.entryType === 'resource') {
          const resource = entry as PerformanceResourceTiming;
          
          // Track resource loading time
          const loadTime = resource.responseEnd - resource.startTime;
          this.recordMetric(`resource_${this.getResourceType(resource.name)}`, loadTime);
          
          // Track resource size
          if (resource.transferSize) {
            this.recordMetric(`resource_size_${this.getResourceType(resource.name)}`, resource.transferSize);
          }
        }
      });
    });
    
    observer.observe({ entryTypes: ['resource'] });
  }
  
  private trackUserInteractions() {
    const interactionTypes = ['click', 'input', 'keydown', 'scroll'];
    
    interactionTypes.forEach(type => {
      document.addEventListener(type, (event) => {
        const startTime = performance.now();
        
        // Use requestIdleCallback to measure interaction responsiveness
        requestIdleCallback(() => {
          const endTime = performance.now();
          this.recordMetric(`interaction_${type}`, endTime - startTime);
        });
      }, { passive: true });
    });
  }
  
  private trackLongTasks() {
    if ('PerformanceObserver' in window) {
      const observer = new PerformanceObserver((list) => {
        list.getEntries().forEach((entry) => {
          this.recordMetric('long_task', entry.duration);
        });
      });
      
      observer.observe({ entryTypes: ['longtask'] });
    }
  }
  
  private trackMemoryUsage() {
    if ('memory' in performance) {
      setInterval(() => {
        const memory = (performance as any).memory;
        this.recordMetric('memory_used', memory.usedJSHeapSize);
        this.recordMetric('memory_total', memory.totalJSHeapSize);
      }, 30000); // Every 30 seconds
    }
  }
  
  private recordMetric(name: string, value: number) {
    this.metrics.set(name, value);
    
    // Send to monitoring service
    this.sendMetric(name, value);
  }
  
  private async sendMetric(name: string, value: number) {
    try {
      await fetch('/api/rum-metrics', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          sessionId: this.sessionId,
          userId: this.userId,
          metric: name,
          value,
          timestamp: Date.now(),
          userAgent: navigator.userAgent,
          url: window.location.href
        })
      });
    } catch (error) {
      console.warn('Failed to send RUM metric:', error);
    }
  }
  
  private getResourceType(url: string): string {
    if (url.match(/\.(css)$/)) return 'css';
    if (url.match(/\.(js)$/)) return 'js';
    if (url.match(/\.(png|jpg|jpeg|gif|webp|svg)$/)) return 'image';
    if (url.match(/\.(woff|woff2|ttf|otf)$/)) return 'font';
    return 'other';
  }
  
  private generateSessionId(): string {
    return `session_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  }
  
  private getCurrentUserId(): string | null {
    try {
      return localStorage.getItem('userId');
    } catch {
      return null;
    }
  }
}

// Initialize RUM
const rum = new RealUserMonitoring();

// Export for use in React components
export const useRUM = () => {
  return {
    trackEvent: (name: string, data?: Record<string, any>) => {
      rum.recordMetric(`custom_${name}`, 1);
    },
    trackTiming: (name: string, duration: number) => {
      rum.recordMetric(`timing_${name}`, duration);
    }
  };
};
```

## 📝 Exercise: Performance Optimization Audit

Conduct a comprehensive performance audit of your application:

### Step 1: Baseline Measurement
- Run Lighthouse audits on key pages
- Measure Core Web Vitals in production
- Analyze bundle sizes and dependencies
- Set up performance monitoring

### Step 2: Optimization Implementation
- Optimize images and fonts
- Implement code splitting
- Add service worker caching
- Optimize JavaScript execution
- Fix layout shifts

### Step 3: Monitoring Setup
- Configure error tracking
- Set up performance budgets
- Implement RUM
- Create performance dashboards

### Step 4: Validation
- Measure improvements
- Compare before/after metrics
- Verify budget compliance
- Document optimizations

**Deliverable:** A complete performance optimization report with before/after metrics, implemented optimizations, and ongoing monitoring setup.

---

**Next:** [Module 7: Security & Accessibility](../07-security-accessibility/) - Master OWASP best practices, XSS prevention, WCAG compliance, and inclusive design principles.