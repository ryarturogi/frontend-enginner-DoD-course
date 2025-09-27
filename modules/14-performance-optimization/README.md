# Module 14: Performance Optimization

## 🎯 Learning Objectives

By the end of this module, you will:
- Master Core Web Vitals optimization techniques
- Implement advanced bundle optimization strategies
- Design efficient caching and resource loading
- Optimize React applications for maximum performance
- Build performance monitoring and regression detection
- Create performance budgets and enforce them in CI/CD

## 🚀 Core Web Vitals Mastery

### Understanding the Metrics

```typescript
// src/lib/web-vitals.ts
export interface WebVitalThresholds {
  lcp: { good: 2500; poor: 4000 };
  fid: { good: 100; poor: 300 };
  cls: { good: 0.1; poor: 0.25 };
  inp: { good: 200; poor: 500 };
  fcp: { good: 1800; poor: 3000 };
  ttfb: { good: 800; poor: 1800 };
}

export const WEB_VITAL_THRESHOLDS: WebVitalThresholds = {
  lcp: { good: 2500, poor: 4000 },
  fid: { good: 100, poor: 300 },
  cls: { good: 0.1, poor: 0.25 },
  inp: { good: 200, poor: 500 },
  fcp: { good: 1800, poor: 3000 },
  ttfb: { good: 800, poor: 1800 },
};

export class WebVitalOptimizer {
  static async optimizeLCP(): Promise<void> {
    // Largest Contentful Paint optimization
    
    // 1. Preload critical resources
    this.preloadCriticalResources();
    
    // 2. Optimize images
    this.optimizeImages();
    
    // 3. Eliminate render-blocking resources
    this.eliminateRenderBlocking();
    
    // 4. Improve server response times
    this.optimizeServerResponse();
  }
  
  private static preloadCriticalResources(): void {
    // Preload critical fonts
    const criticalFonts = [
      '/fonts/inter-var.woff2',
      '/fonts/inter-bold.woff2',
    ];
    
    criticalFonts.forEach(font => {
      const link = document.createElement('link');
      link.rel = 'preload';
      link.href = font;
      link.as = 'font';
      link.type = 'font/woff2';
      link.crossOrigin = 'anonymous';
      document.head.appendChild(link);
    });
    
    // Preload hero images
    const heroImages = document.querySelectorAll('[data-preload="true"]');
    heroImages.forEach(img => {
      if (img instanceof HTMLImageElement) {
        const link = document.createElement('link');
        link.rel = 'preload';
        link.href = img.src;
        link.as = 'image';
        document.head.appendChild(link);
      }
    });
  }
  
  private static optimizeImages(): void {
    // Implement responsive images with srcset
    const images = document.querySelectorAll('img[data-optimize="true"]');
    
    images.forEach(img => {
      if (img instanceof HTMLImageElement) {
        // Add srcset for responsive images
        const src = img.src;
        const baseName = src.replace(/\.[^/.]+$/, '');
        const extension = src.split('.').pop();
        
        img.srcset = [
          `${baseName}-320w.${extension} 320w`,
          `${baseName}-640w.${extension} 640w`,
          `${baseName}-1024w.${extension} 1024w`,
          `${baseName}-1440w.${extension} 1440w`,
        ].join(', ');
        
        img.sizes = '(max-width: 320px) 280px, (max-width: 640px) 600px, (max-width: 1024px) 980px, 1400px';
        
        // Add loading="lazy" for below-fold images
        const rect = img.getBoundingClientRect();
        if (rect.top > window.innerHeight) {
          img.loading = 'lazy';
        }
      }
    });
  }
  
  static optimizeFID(): void {
    // First Input Delay optimization
    
    // 1. Code splitting
    this.implementCodeSplitting();
    
    // 2. Reduce main thread blocking
    this.reduceMainThreadBlocking();
    
    // 3. Optimize third-party scripts
    this.optimizeThirdPartyScripts();
  }
  
  private static implementCodeSplitting(): void {
    // Dynamic imports for route-based splitting
    const routeComponents = {
      '/dashboard': () => import('../pages/Dashboard'),
      '/profile': () => import('../pages/Profile'),
      '/settings': () => import('../pages/Settings'),
    };
    
    // Component-based splitting
    const heavyComponents = {
      'DataVisualization': () => import('../components/DataVisualization'),
      'RichTextEditor': () => import('../components/RichTextEditor'),
      'VideoPlayer': () => import('../components/VideoPlayer'),
    };
  }
  
  private static reduceMainThreadBlocking(): void {
    // Use Web Workers for heavy computations
    const worker = new Worker('/workers/calculation-worker.js');
    
    // Implement time slicing for large data processing
    const processLargeDataset = async (data: any[]) => {
      const CHUNK_SIZE = 100;
      const results = [];
      
      for (let i = 0; i < data.length; i += CHUNK_SIZE) {
        const chunk = data.slice(i, i + CHUNK_SIZE);
        const chunkResults = chunk.map(processItem);
        results.push(...chunkResults);
        
        // Yield control back to browser
        await new Promise(resolve => setTimeout(resolve, 0));
      }
      
      return results;
    };
  }
  
  static optimizeCLS(): void {
    // Cumulative Layout Shift optimization
    
    // 1. Set size attributes for media
    this.setSizeAttributesForMedia();
    
    // 2. Reserve space for dynamic content
    this.reserveSpaceForDynamicContent();
    
    // 3. Avoid inserting content above existing content
    this.avoidContentInsertion();
  }
  
  private static setSizeAttributesForMedia(): void {
    // Ensure all images have width and height attributes
    const images = document.querySelectorAll('img:not([width]):not([height])');
    images.forEach(img => {
      if (img instanceof HTMLImageElement) {
        // Calculate aspect ratio and set dimensions
        const aspectRatio = img.naturalWidth / img.naturalHeight;
        const containerWidth = img.parentElement?.clientWidth || 300;
        
        img.width = containerWidth;
        img.height = containerWidth / aspectRatio;
      }
    });
  }
  
  private static reserveSpaceForDynamicContent(): void {
    // Use CSS to reserve space for dynamic content
    const style = document.createElement('style');
    style.textContent = `
      .skeleton-loader {
        background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
        background-size: 200% 100%;
        animation: loading 1.5s infinite;
      }
      
      @keyframes loading {
        0% { background-position: 200% 0; }
        100% { background-position: -200% 0; }
      }
      
      .ad-placeholder {
        min-height: 250px;
        background: #f5f5f5;
        border: 1px dashed #ccc;
      }
    `;
    document.head.appendChild(style);
  }
}
```

### Advanced React Performance Optimization

```typescript
// src/hooks/performance/useOptimizedState.ts
export const useOptimizedState = <T>(
  initialState: T,
  options?: {
    debounceMs?: number;
    throttleMs?: number;
    compareFn?: (prev: T, next: T) => boolean;
  }
): [T, (newState: T | ((prev: T) => T)) => void] => {
  const [state, setState] = useState<T>(initialState);
  const { debounceMs, throttleMs, compareFn = Object.is } = options || {};
  
  const debouncedSetState = useMemo(() => {
    let timeoutId: NodeJS.Timeout;
    
    return (newState: T | ((prev: T) => T)) => {
      if (timeoutId) clearTimeout(timeoutId);
      
      timeoutId = setTimeout(() => {
        setState(prev => {
          const next = typeof newState === 'function' 
            ? (newState as (prev: T) => T)(prev)
            : newState;
          
          return compareFn(prev, next) ? prev : next;
        });
      }, debounceMs || 0);
    };
  }, [debounceMs, compareFn]);
  
  const throttledSetState = useMemo(() => {
    let lastUpdate = 0;
    
    return (newState: T | ((prev: T) => T)) => {
      const now = Date.now();
      
      if (now - lastUpdate >= (throttleMs || 0)) {
        lastUpdate = now;
        setState(prev => {
          const next = typeof newState === 'function' 
            ? (newState as (prev: T) => T)(prev)
            : newState;
          
          return compareFn(prev, next) ? prev : next;
        });
      }
    };
  }, [throttleMs, compareFn]);
  
  const optimizedSetState = debounceMs 
    ? debouncedSetState 
    : throttleMs 
      ? throttledSetState 
      : setState;
  
  return [state, optimizedSetState];
};

// src/components/optimized/VirtualizedList.tsx
interface VirtualizedListProps<T> {
  items: T[];
  itemHeight: number;
  containerHeight: number;
  renderItem: (item: T, index: number) => React.ReactNode;
  overscan?: number;
}

export const VirtualizedList = <T,>({
  items,
  itemHeight,
  containerHeight,
  renderItem,
  overscan = 5,
}: VirtualizedListProps<T>) => {
  const [scrollTop, setScrollTop] = useOptimizedState(0, { throttleMs: 16 });
  
  const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
  const endIndex = Math.min(
    items.length - 1,
    Math.floor((scrollTop + containerHeight) / itemHeight) + overscan
  );
  
  const visibleItems = useMemo(() => {
    return items.slice(startIndex, endIndex + 1).map((item, index) => ({
      item,
      index: startIndex + index,
    }));
  }, [items, startIndex, endIndex]);
  
  const totalHeight = items.length * itemHeight;
  const offsetY = startIndex * itemHeight;
  
  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.currentTarget.scrollTop)}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map(({ item, index }) => (
            <div
              key={index}
              style={{ height: itemHeight }}
            >
              {renderItem(item, index)}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

// src/hooks/performance/useIntersectionObserver.ts
export const useIntersectionObserver = (
  options?: IntersectionObserverInit
) => {
  const [entries, setEntries] = useState<IntersectionObserverEntry[]>([]);
  const observer = useRef<IntersectionObserver>();
  
  const observe = useCallback((element: Element) => {
    if (!observer.current) {
      observer.current = new IntersectionObserver((entries) => {
        setEntries(entries);
      }, options);
    }
    
    observer.current.observe(element);
    
    return () => {
      observer.current?.unobserve(element);
    };
  }, [options]);
  
  useEffect(() => {
    return () => {
      observer.current?.disconnect();
    };
  }, []);
  
  return { entries, observe };
};

// src/components/optimized/LazyImage.tsx
interface LazyImageProps extends React.ImgHTMLAttributes<HTMLImageElement> {
  fallback?: React.ReactNode;
  threshold?: number;
}

export const LazyImage: React.FC<LazyImageProps> = ({
  src,
  fallback,
  threshold = 0.1,
  ...props
}) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);
  const imgRef = useRef<HTMLImageElement>(null);
  
  const { observe } = useIntersectionObserver({
    threshold,
    rootMargin: '50px',
  });
  
  useEffect(() => {
    if (imgRef.current) {
      const unobserve = observe(imgRef.current);
      return unobserve;
    }
  }, [observe]);
  
  useEffect(() => {
    if (isInView && src && !isLoaded) {
      const img = new Image();
      img.onload = () => setIsLoaded(true);
      img.src = src;
    }
  }, [isInView, src, isLoaded]);
  
  return (
    <div ref={imgRef} className="lazy-image-container">
      {isLoaded ? (
        <img
          src={src}
          {...props}
          onLoad={() => setIsLoaded(true)}
        />
      ) : (
        fallback || <div className="image-placeholder" />
      )}
    </div>
  );
};
```

## 📦 Bundle Optimization

### Advanced Webpack Configuration

```typescript
// webpack.config.js
const path = require('path');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  // Bundle splitting strategy
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor libraries that rarely change
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          enforce: true,
        },
        
        // Common utilities used across features
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true,
        },
        
        // React and related libraries
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router)[\\/]/,
          name: 'react',
          chunks: 'all',
        },
        
        // UI component libraries
        ui: {
          test: /[\\/]node_modules[\\/](@mui|@mantine|antd)[\\/]/,
          name: 'ui-library',
          chunks: 'all',
        },
        
        // Analytics and monitoring
        analytics: {
          test: /[\\/]node_modules[\\/](@sentry|logrocket|hotjar)[\\/]/,
          name: 'analytics',
          chunks: 'all',
        },
      },
    },
    
    // Minimize bundle size
    usedExports: true,
    sideEffects: false,
    
    // Runtime chunk for webpack module loading
    runtimeChunk: {
      name: 'runtime',
    },
  },
  
  // Module resolution optimization
  resolve: {
    // Optimize module resolution
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],
    
    // Prefer ES modules for better tree shaking
    mainFields: ['module', 'main'],
    
    // Alias for commonly used modules
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      
      // Replace heavy libraries with lighter alternatives
      'moment': 'dayjs',
      'lodash': 'lodash-es',
    },
  },
  
  // Production optimizations
  ...(process.env.NODE_ENV === 'production' && {
    plugins: [
      // Analyze bundle size
      new BundleAnalyzerPlugin({
        analyzerMode: 'static',
        openAnalyzer: false,
        reportFilename: 'bundle-report.html',
      }),
      
      // Compression
      new CompressionPlugin({
        algorithm: 'gzip',
        test: /\.(js|css|html|svg)$/,
        threshold: 8192,
        minRatio: 0.8,
      }),
    ],
  }),
};

// next.config.js for Next.js projects
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Experimental features for better performance
  experimental: {
    // App directory for better performance
    appDir: true,
    
    // Server components by default
    serverComponents: true,
    
    // Optimize images
    optimizeImages: true,
    
    // Bundle analyzer in development
    bundlePagesExternals: true,
  },
  
  // Image optimization
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  
  // Compression
  compress: true,
  
  // Bundle optimization
  webpack: (config, { dev, isServer }) => {
    // Production optimizations
    if (!dev && !isServer) {
      // Tree shaking optimization
      config.optimization.usedExports = true;
      config.optimization.sideEffects = false;
      
      // Remove unused CSS
      config.plugins.push(
        new (require('@fullhuman/postcss-purgecss'))({
          content: ['./src/**/*.{js,jsx,ts,tsx}'],
          defaultExtractor: content => content.match(/[\w-/:]+(?<!:)/g) || [],
        })
      );
    }
    
    return config;
  },
  
  // Headers for caching
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block',
          },
        ],
      },
      {
        source: '/static/(.*)',
        headers: [
          {
            key: 'Cache-Control',
            value: 'public, max-age=31536000, immutable',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

### Tree Shaking and Dead Code Elimination

```typescript
// src/utils/tree-shaking-optimizer.ts

// ❌ Bad: Imports entire library
import _ from 'lodash';
import * as icons from 'react-icons/fa';
import moment from 'moment';

// ✅ Good: Import only what you need
import { debounce, throttle } from 'lodash-es';
import { FaUser, FaHome } from 'react-icons/fa';
import dayjs from 'dayjs';

// Optimize library imports
export class TreeShakingOptimizer {
  // ✅ Use babel-plugin-import for automatic optimization
  static configureOptimizedImports() {
    // .babelrc.js
    return {
      plugins: [
        [
          'import',
          {
            libraryName: 'antd',
            style: 'css',
          },
          'antd',
        ],
        [
          'import',
          {
            libraryName: 'lodash',
            libraryDirectory: '',
            camel2DashComponentName: false,
          },
          'lodash',
        ],
      ],
    };
  }
  
  // ✅ Create barrel exports that support tree shaking
  static createOptimizedBarrelExports() {
    // src/components/index.ts
    // ❌ Bad: Re-exports everything
    // export * from './Button';
    // export * from './Input';
    
    // ✅ Good: Named exports only
    export { Button } from './Button';
    export { Input } from './Input';
    export type { ButtonProps } from './Button';
    export type { InputProps } from './Input';
  }
  
  // ✅ Use dynamic imports for conditional features
  static async loadFeatureConditionally(featureName: string) {
    switch (featureName) {
      case 'chart':
        const { Chart } = await import('./Chart');
        return Chart;
      
      case 'editor':
        const { RichTextEditor } = await import('./RichTextEditor');
        return RichTextEditor;
      
      case 'datePicker':
        const { DatePicker } = await import('./DatePicker');
        return DatePicker;
      
      default:
        return null;
    }
  }
}

// Utility for measuring bundle impact
export class BundleAnalyzer {
  static measureImportImpact(importName: string) {
    const before = performance.memory?.usedJSHeapSize || 0;
    
    return {
      start: () => {
        console.time(`Import: ${importName}`);
      },
      
      end: () => {
        console.timeEnd(`Import: ${importName}`);
        const after = performance.memory?.usedJSHeapSize || 0;
        const impact = after - before;
        
        console.log(`Memory impact of ${importName}: ${impact} bytes`);
        
        return impact;
      },
    };
  }
  
  static analyzeDependencySize() {
    // This would integrate with webpack-bundle-analyzer data
    const dependencies = {
      'react': { size: 42000, gzipped: 13200 },
      'react-dom': { size: 130000, gzipped: 42000 },
      'lodash': { size: 70000, gzipped: 25000 },
      'moment': { size: 67000, gzipped: 16900 },
      'dayjs': { size: 2800, gzipped: 1200 },
    };
    
    return dependencies;
  }
}
```

## 🗄️ Caching Strategies

### Multi-Level Caching System

```typescript
// src/lib/caching/cache-manager.ts
export enum CacheLevel {
  MEMORY = 'memory',
  SESSION_STORAGE = 'sessionStorage',
  LOCAL_STORAGE = 'localStorage',
  INDEXED_DB = 'indexedDB',
  SERVICE_WORKER = 'serviceWorker',
}

export interface CacheConfig {
  level: CacheLevel;
  ttl?: number; // Time to live in milliseconds
  maxSize?: number; // Maximum cache size
  compression?: boolean;
  encryption?: boolean;
}

export class CacheManager {
  private memoryCache = new Map<string, { data: any; expires: number }>();
  private maxMemorySize = 100; // MB
  
  async set(
    key: string, 
    data: any, 
    config: CacheConfig = { level: CacheLevel.MEMORY }
  ): Promise<void> {
    const expires = config.ttl ? Date.now() + config.ttl : Infinity;
    const entry = { data, expires };
    
    switch (config.level) {
      case CacheLevel.MEMORY:
        await this.setMemoryCache(key, entry);
        break;
        
      case CacheLevel.SESSION_STORAGE:
        await this.setSessionStorage(key, entry, config);
        break;
        
      case CacheLevel.LOCAL_STORAGE:
        await this.setLocalStorage(key, entry, config);
        break;
        
      case CacheLevel.INDEXED_DB:
        await this.setIndexedDB(key, entry, config);
        break;
        
      case CacheLevel.SERVICE_WORKER:
        await this.setServiceWorkerCache(key, entry, config);
        break;
    }
  }
  
  async get(key: string, level: CacheLevel = CacheLevel.MEMORY): Promise<any> {
    switch (level) {
      case CacheLevel.MEMORY:
        return this.getMemoryCache(key);
        
      case CacheLevel.SESSION_STORAGE:
        return this.getSessionStorage(key);
        
      case CacheLevel.LOCAL_STORAGE:
        return this.getLocalStorage(key);
        
      case CacheLevel.INDEXED_DB:
        return this.getIndexedDB(key);
        
      case CacheLevel.SERVICE_WORKER:
        return this.getServiceWorkerCache(key);
        
      default:
        return null;
    }
  }
  
  private async setMemoryCache(key: string, entry: any): Promise<void> {
    // Check memory usage
    if (this.memoryCache.size > 1000) {
      this.evictOldestEntries();
    }
    
    this.memoryCache.set(key, entry);
  }
  
  private getMemoryCache(key: string): any {
    const entry = this.memoryCache.get(key);
    
    if (!entry) return null;
    
    if (entry.expires < Date.now()) {
      this.memoryCache.delete(key);
      return null;
    }
    
    return entry.data;
  }
  
  private async setLocalStorage(
    key: string, 
    entry: any, 
    config: CacheConfig
  ): Promise<void> {
    try {
      let data = JSON.stringify(entry);
      
      if (config.compression) {
        data = await this.compress(data);
      }
      
      if (config.encryption) {
        data = await this.encrypt(data);
      }
      
      localStorage.setItem(`cache:${key}`, data);
    } catch (error) {
      console.warn('Failed to set localStorage cache:', error);
    }
  }
  
  private getLocalStorage(key: string): any {
    try {
      const cached = localStorage.getItem(`cache:${key}`);
      if (!cached) return null;
      
      let data = cached;
      
      // Handle decompression and decryption if needed
      // This would be implemented based on your compression/encryption strategy
      
      const entry = JSON.parse(data);
      
      if (entry.expires < Date.now()) {
        localStorage.removeItem(`cache:${key}`);
        return null;
      }
      
      return entry.data;
    } catch (error) {
      console.warn('Failed to get localStorage cache:', error);
      return null;
    }
  }
  
  private async setIndexedDB(
    key: string, 
    entry: any, 
    config: CacheConfig
  ): Promise<void> {
    const db = await this.getIndexedDB();
    const transaction = db.transaction(['cache'], 'readwrite');
    const store = transaction.objectStore('cache');
    
    await store.put({
      key,
      data: entry.data,
      expires: entry.expires,
      created: Date.now(),
    });
  }
  
  private async getIndexedDB(key: string): Promise<any> {
    const db = await this.getIndexedDB();
    const transaction = db.transaction(['cache'], 'readonly');
    const store = transaction.objectStore('cache');
    
    const result = await store.get(key);
    
    if (!result || result.expires < Date.now()) {
      // Clean up expired entry
      if (result) {
        const deleteTransaction = db.transaction(['cache'], 'readwrite');
        const deleteStore = deleteTransaction.objectStore('cache');
        await deleteStore.delete(key);
      }
      return null;
    }
    
    return result.data;
  }
  
  private async getIndexedDB(): Promise<IDBDatabase> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open('AppCache', 1);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
      
      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        if (!db.objectStoreNames.contains('cache')) {
          db.createObjectStore('cache', { keyPath: 'key' });
        }
      };
    });
  }
  
  private evictOldestEntries(): void {
    const entries = Array.from(this.memoryCache.entries());
    const sortedEntries = entries.sort((a, b) => a[1].expires - b[1].expires);
    
    // Remove oldest 10% of entries
    const removeCount = Math.floor(entries.length * 0.1);
    for (let i = 0; i < removeCount; i++) {
      this.memoryCache.delete(sortedEntries[i][0]);
    }
  }
  
  private async compress(data: string): Promise<string> {
    // Implement compression (e.g., using CompressionStream API)
    return data; // Placeholder
  }
  
  private async encrypt(data: string): Promise<string> {
    // Implement encryption for sensitive data
    return data; // Placeholder
  }
}

// Smart caching hook
export const useSmartCache = <T>(
  key: string,
  fetcher: () => Promise<T>,
  config?: CacheConfig
) => {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const cacheManager = useMemo(() => new CacheManager(), []);
  
  const fetchData = useCallback(async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      // Try cache first
      const cached = await cacheManager.get(key, config?.level);
      if (cached) {
        setData(cached);
        setIsLoading(false);
        return cached;
      }
      
      // Fetch fresh data
      const freshData = await fetcher();
      
      // Cache the result
      await cacheManager.set(key, freshData, config);
      
      setData(freshData);
      return freshData;
    } catch (err) {
      setError(err as Error);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, [key, fetcher, cacheManager, config]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return {
    data,
    isLoading,
    error,
    refetch: fetchData,
    invalidate: () => cacheManager.delete(key),
  };
};
```

### Service Worker Caching

```typescript
// public/sw.js - Service Worker for advanced caching
const CACHE_NAME = 'app-cache-v1';
const RUNTIME_CACHE = 'runtime-cache-v1';

// Cache strategies
const CACHE_STRATEGIES = {
  CACHE_FIRST: 'cache-first',
  NETWORK_FIRST: 'network-first',
  STALE_WHILE_REVALIDATE: 'stale-while-revalidate',
  NETWORK_ONLY: 'network-only',
  CACHE_ONLY: 'cache-only',
};

// Resource categorization
const CACHE_RULES = [
  {
    pattern: /\.(?:png|jpg|jpeg|svg|gif|webp|avif)$/,
    strategy: CACHE_STRATEGIES.CACHE_FIRST,
    cacheName: 'images',
    maxEntries: 100,
    maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
  },
  {
    pattern: /\.(?:js|css)$/,
    strategy: CACHE_STRATEGIES.STALE_WHILE_REVALIDATE,
    cacheName: 'static-resources',
    maxEntries: 50,
    maxAgeSeconds: 7 * 24 * 60 * 60, // 7 days
  },
  {
    pattern: /^https:\/\/api\./,
    strategy: CACHE_STRATEGIES.NETWORK_FIRST,
    cacheName: 'api-cache',
    maxEntries: 100,
    maxAgeSeconds: 5 * 60, // 5 minutes
  },
  {
    pattern: /^https:\/\/fonts\./,
    strategy: CACHE_STRATEGIES.CACHE_FIRST,
    cacheName: 'fonts',
    maxEntries: 20,
    maxAgeSeconds: 365 * 24 * 60 * 60, // 1 year
  },
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll([
        '/',
        '/static/js/bundle.js',
        '/static/css/main.css',
        '/manifest.json',
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  const request = event.request;
  const url = new URL(request.url);
  
  // Skip non-GET requests
  if (request.method !== 'GET') return;
  
  // Find matching cache rule
  const rule = CACHE_RULES.find(rule => rule.pattern.test(request.url));
  
  if (rule) {
    event.respondWith(handleRequest(request, rule));
  }
});

async function handleRequest(request, rule) {
  const cache = await caches.open(rule.cacheName);
  
  switch (rule.strategy) {
    case CACHE_STRATEGIES.CACHE_FIRST:
      return cacheFirst(request, cache, rule);
    
    case CACHE_STRATEGIES.NETWORK_FIRST:
      return networkFirst(request, cache, rule);
    
    case CACHE_STRATEGIES.STALE_WHILE_REVALIDATE:
      return staleWhileRevalidate(request, cache, rule);
    
    default:
      return fetch(request);
  }
}

async function cacheFirst(request, cache, rule) {
  const cached = await cache.match(request);
  
  if (cached && !isExpired(cached, rule.maxAgeSeconds)) {
    return cached;
  }
  
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      cache.put(request, response.clone());
      cleanupCache(cache, rule.maxEntries);
    }
    
    return response;
  } catch (error) {
    return cached || new Response('Offline', { status: 503 });
  }
}

async function networkFirst(request, cache, rule) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      cache.put(request, response.clone());
      cleanupCache(cache, rule.maxEntries);
    }
    
    return response;
  } catch (error) {
    const cached = await cache.match(request);
    return cached || new Response('Offline', { status: 503 });
  }
}

async function staleWhileRevalidate(request, cache, rule) {
  const cached = await cache.match(request);
  
  // Return cached version immediately if available
  const response = cached || fetch(request);
  
  // Update cache in background
  const updatePromise = fetch(request).then(freshResponse => {
    if (freshResponse.ok) {
      cache.put(request, freshResponse.clone());
      cleanupCache(cache, rule.maxEntries);
    }
    return freshResponse;
  }).catch(() => {
    // Ignore network errors for background updates
  });
  
  return cached ? response : updatePromise;
}

function isExpired(response, maxAgeSeconds) {
  const dateHeader = response.headers.get('date');
  if (!dateHeader) return true;
  
  const responseDate = new Date(dateHeader);
  const now = new Date();
  const ageInSeconds = (now.getTime() - responseDate.getTime()) / 1000;
  
  return ageInSeconds > maxAgeSeconds;
}

async function cleanupCache(cache, maxEntries) {
  const keys = await cache.keys();
  
  if (keys.length <= maxEntries) return;
  
  // Remove oldest entries
  const entriesToDelete = keys.length - maxEntries;
  const oldestKeys = keys.slice(0, entriesToDelete);
  
  await Promise.all(oldestKeys.map(key => cache.delete(key)));
}

// Background sync for offline actions
self.addEventListener('sync', (event) => {
  if (event.tag === 'background-sync') {
    event.waitUntil(processOfflineActions());
  }
});

async function processOfflineActions() {
  // Process queued offline actions
  const offlineActions = await getOfflineActions();
  
  for (const action of offlineActions) {
    try {
      await fetch(action.url, action.options);
      await removeOfflineAction(action.id);
    } catch (error) {
      console.log('Failed to sync action:', action);
    }
  }
}
```

## 📊 Performance Monitoring & Budgets

### Performance Budget Framework

```typescript
// src/lib/performance/budget-monitor.ts
export interface PerformanceBudget {
  metrics: {
    // Core Web Vitals
    lcp: number;    // Largest Contentful Paint (ms)
    fid: number;    // First Input Delay (ms)
    cls: number;    // Cumulative Layout Shift (score)
    inp: number;    // Interaction to Next Paint (ms)
    fcp: number;    // First Contentful Paint (ms)
    ttfb: number;   // Time to First Byte (ms)
  };
  
  resources: {
    // Bundle sizes
    initialBundle: number;     // bytes
    totalJavaScript: number;   // bytes
    totalCSS: number;          // bytes
    totalImages: number;       // bytes
    totalFonts: number;        // bytes
  };
  
  network: {
    // Network requests
    totalRequests: number;
    criticalRequests: number;
    thirdPartyRequests: number;
  };
  
  runtime: {
    // Runtime performance
    memoryUsage: number;       // bytes
    mainThreadBlocking: number; // ms
    longTasks: number;         // count
  };
}

export class BudgetMonitor {
  private budget: PerformanceBudget;
  private violations: string[] = [];
  
  constructor(budget: PerformanceBudget) {
    this.budget = budget;
  }
  
  async checkBudget(): Promise<BudgetResult> {
    this.violations = [];
    
    // Check Core Web Vitals
    await this.checkWebVitals();
    
    // Check resource sizes
    await this.checkResourceSizes();
    
    // Check network requests
    await this.checkNetworkRequests();
    
    // Check runtime performance
    await this.checkRuntimePerformance();
    
    return {
      passed: this.violations.length === 0,
      violations: this.violations,
      score: this.calculateScore(),
      report: this.generateReport(),
    };
  }
  
  private async checkWebVitals(): Promise<void> {
    const vitals = await this.getCurrentWebVitals();
    
    if (vitals.lcp > this.budget.metrics.lcp) {
      this.violations.push(`LCP exceeded budget: ${vitals.lcp}ms > ${this.budget.metrics.lcp}ms`);
    }
    
    if (vitals.fid > this.budget.metrics.fid) {
      this.violations.push(`FID exceeded budget: ${vitals.fid}ms > ${this.budget.metrics.fid}ms`);
    }
    
    if (vitals.cls > this.budget.metrics.cls) {
      this.violations.push(`CLS exceeded budget: ${vitals.cls} > ${this.budget.metrics.cls}`);
    }
    
    if (vitals.inp > this.budget.metrics.inp) {
      this.violations.push(`INP exceeded budget: ${vitals.inp}ms > ${this.budget.metrics.inp}ms`);
    }
  }
  
  private async checkResourceSizes(): Promise<void> {
    const resources = await this.getCurrentResourceSizes();
    
    if (resources.initialBundle > this.budget.resources.initialBundle) {
      this.violations.push(`Initial bundle size exceeded: ${this.formatBytes(resources.initialBundle)} > ${this.formatBytes(this.budget.resources.initialBundle)}`);
    }
    
    if (resources.totalJavaScript > this.budget.resources.totalJavaScript) {
      this.violations.push(`Total JavaScript size exceeded: ${this.formatBytes(resources.totalJavaScript)} > ${this.formatBytes(this.budget.resources.totalJavaScript)}`);
    }
    
    if (resources.totalCSS > this.budget.resources.totalCSS) {
      this.violations.push(`Total CSS size exceeded: ${this.formatBytes(resources.totalCSS)} > ${this.formatBytes(this.budget.resources.totalCSS)}`);
    }
  }
  
  private async getCurrentWebVitals(): Promise<any> {
    // Integration with web-vitals library
    return new Promise((resolve) => {
      import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB, onINP }) => {
        const vitals: any = {};
        
        getCLS((metric) => { vitals.cls = metric.value; });
        getFID((metric) => { vitals.fid = metric.value; });
        getFCP((metric) => { vitals.fcp = metric.value; });
        getLCP((metric) => { vitals.lcp = metric.value; });
        getTTFB((metric) => { vitals.ttfb = metric.value; });
        onINP((metric) => { vitals.inp = metric.value; });
        
        // Wait for metrics to be collected
        setTimeout(() => resolve(vitals), 1000);
      });
    });
  }
  
  private async getCurrentResourceSizes(): Promise<any> {
    const resources = performance.getEntriesByType('resource') as PerformanceResourceTiming[];
    
    const sizes = {
      initialBundle: 0,
      totalJavaScript: 0,
      totalCSS: 0,
      totalImages: 0,
      totalFonts: 0,
    };
    
    resources.forEach((resource) => {
      const size = resource.transferSize || resource.encodedBodySize || 0;
      
      if (resource.name.includes('.js')) {
        sizes.totalJavaScript += size;
        
        if (resource.name.includes('main') || resource.name.includes('bundle')) {
          sizes.initialBundle += size;
        }
      } else if (resource.name.includes('.css')) {
        sizes.totalCSS += size;
      } else if (/\.(png|jpg|jpeg|gif|webp|svg)$/.test(resource.name)) {
        sizes.totalImages += size;
      } else if (/\.(woff|woff2|ttf|otf)$/.test(resource.name)) {
        sizes.totalFonts += size;
      }
    });
    
    return sizes;
  }
  
  private formatBytes(bytes: number): string {
    if (bytes === 0) return '0 Bytes';
    
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }
  
  private calculateScore(): number {
    const totalChecks = Object.keys(this.budget.metrics).length + 
                       Object.keys(this.budget.resources).length +
                       Object.keys(this.budget.network).length +
                       Object.keys(this.budget.runtime).length;
    
    const passedChecks = totalChecks - this.violations.length;
    return Math.round((passedChecks / totalChecks) * 100);
  }
  
  private generateReport(): string {
    return `
# Performance Budget Report

## Status: ${this.violations.length === 0 ? '✅ PASSED' : '❌ FAILED'}
## Score: ${this.calculateScore()}/100

### Violations (${this.violations.length})
${this.violations.map(v => `- ${v}`).join('\n')}

### Budget Thresholds
- **LCP**: ${this.budget.metrics.lcp}ms
- **FID**: ${this.budget.metrics.fid}ms
- **CLS**: ${this.budget.metrics.cls}
- **Initial Bundle**: ${this.formatBytes(this.budget.resources.initialBundle)}
- **Total JavaScript**: ${this.formatBytes(this.budget.resources.totalJavaScript)}

Generated at: ${new Date().toISOString()}
    `.trim();
  }
}

// CI/CD Integration
export class BudgetEnforcer {
  static async enforceBudgetInCI(): Promise<void> {
    const budget: PerformanceBudget = {
      metrics: {
        lcp: 2500,  // 2.5s
        fid: 100,   // 100ms
        cls: 0.1,   // 0.1
        inp: 200,   // 200ms
        fcp: 1800,  // 1.8s
        ttfb: 800,  // 800ms
      },
      resources: {
        initialBundle: 500 * 1024,      // 500KB
        totalJavaScript: 1024 * 1024,   // 1MB
        totalCSS: 200 * 1024,           // 200KB
        totalImages: 2 * 1024 * 1024,   // 2MB
        totalFonts: 100 * 1024,         // 100KB
      },
      network: {
        totalRequests: 50,
        criticalRequests: 10,
        thirdPartyRequests: 5,
      },
      runtime: {
        memoryUsage: 50 * 1024 * 1024,  // 50MB
        mainThreadBlocking: 1000,        // 1s
        longTasks: 5,                    // 5 tasks
      },
    };
    
    const monitor = new BudgetMonitor(budget);
    const result = await monitor.checkBudget();
    
    // Write report to file
    const fs = require('fs');
    fs.writeFileSync('performance-budget-report.md', result.report);
    
    // Exit with error code if budget failed
    if (!result.passed) {
      console.error('❌ Performance budget check failed!');
      console.error(result.report);
      process.exit(1);
    }
    
    console.log('✅ Performance budget check passed!');
    console.log(`Score: ${result.score}/100`);
  }
}
```

## 🎯 Implementation Exercise

Create a comprehensive performance optimization system:

### Phase 1: Core Web Vitals Optimization
1. Implement LCP optimization techniques
2. Set up FID/INP monitoring and optimization
3. Create CLS measurement and fixes
4. Build real-time performance dashboard

### Phase 2: Bundle Optimization
1. Configure advanced webpack splitting
2. Implement tree shaking optimization
3. Set up dynamic imports for features
4. Create bundle analysis automation

### Phase 3: Caching Implementation
1. Build multi-level cache system
2. Implement service worker caching
3. Create smart cache invalidation
4. Set up cache performance monitoring

### Phase 4: Performance Monitoring
1. Create performance budget framework
2. Set up automated budget enforcement
3. Build performance regression detection
4. Implement CI/CD performance gates

**Deliverable:** A complete performance optimization system with monitoring, budgets, and automated enforcement.

---

**Next:** [Module 15: Interview Preparation](../15-interview-preparation/) - Prepare for senior frontend engineer interviews with comprehensive practice materials.