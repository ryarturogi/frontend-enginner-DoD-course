# Module 12: CI/CD & DevOps Awareness

## 🎯 Learning Objectives

By the end of this module, you will:
- Build comprehensive CI/CD pipelines with automated quality gates
- Implement progressive deployment strategies with feature flags
- Set up automated testing, linting, and security scanning in pipelines
- Configure containerized deployments for frontend applications
- Master environment management and configuration strategies
- Understand monitoring and alerting for deployment health

## 🔄 Comprehensive CI/CD Pipeline Architecture

### Pipeline Overview

```mermaid
graph LR
    A[Code Push] --> B[Build Stage]
    B --> C[Test Stage]
    C --> D[Security Stage]
    D --> E[Quality Gate]
    E --> F[Package Stage]
    F --> G[Deploy Staging]
    G --> H[E2E Tests]
    H --> I[Manual Approval]
    I --> J[Deploy Production]
    J --> K[Smoke Tests]
    K --> L[Monitor]
```

### Multi-Stage Pipeline Implementation

```yaml
# .github/workflows/main.yml
name: Production CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Stage 1: Build and Basic Validation
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
      should-deploy: ${{ steps.check.outputs.should-deploy }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Full history for conventional commits
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Generate version
        id: version
        run: |
          if [[ ${{ github.ref }} == 'refs/heads/main' ]]; then
            VERSION=$(npx semantic-release --dry-run | grep -o 'next version is [0-9]\+\.[0-9]\+\.[0-9]\+' | cut -d' ' -f4)
          else
            VERSION="${{ github.sha }}"
          fi
          echo "version=${VERSION}" >> $GITHUB_OUTPUT
      
      - name: Type checking
        run: pnpm run type-check
      
      - name: Build application
        run: pnpm run build
        env:
          NODE_ENV: production
          VITE_VERSION: ${{ steps.version.outputs.version }}
      
      - name: Cache build artifacts
        uses: actions/cache@v3
        with:
          path: |
            dist/
            .next/
            build/
          key: build-${{ github.sha }}
      
      - name: Check deployment conditions
        id: check
        run: |
          if [[ ${{ github.ref }} == 'refs/heads/main' || ${{ github.ref }} == 'refs/heads/develop' ]]; then
            echo "should-deploy=true" >> $GITHUB_OUTPUT
          else
            echo "should-deploy=false" >> $GITHUB_OUTPUT
          fi

  # Stage 2: Code Quality and Linting
  code-quality:
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: ESLint
        run: |
          pnpm run lint --format=@microsoft/eslint-formatter-sarif --output-file=eslint-results.sarif
        continue-on-error: true
      
      - name: Upload ESLint results to GitHub
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: eslint-results.sarif
          wait-for-processing: true
      
      - name: Prettier check
        run: pnpm run format:check
      
      - name: Check for TODO/FIXME comments
        run: |
          if grep -r "TODO\|FIXME" src/ --exclude-dir=node_modules; then
            echo "::warning::TODO/FIXME comments found"
          fi
      
      - name: Dependency audit
        run: pnpm audit --audit-level moderate
      
      - name: Check bundle size
        run: |
          pnpm run build
          npx bundlesize

  # Stage 3: Automated Testing
  test:
    runs-on: ubuntu-latest
    needs: build
    strategy:
      matrix:
        test-type: [unit, integration, accessibility]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run unit tests
        if: matrix.test-type == 'unit'
        run: |
          pnpm run test:coverage --reporter=default --reporter=github-actions
        env:
          CI: true
      
      - name: Run integration tests
        if: matrix.test-type == 'integration'
        run: pnpm run test:integration
        env:
          CI: true
      
      - name: Run accessibility tests
        if: matrix.test-type == 'accessibility'
        run: pnpm run test:a11y
      
      - name: Upload coverage to Codecov
        if: matrix.test-type == 'unit'
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage/lcov.info
          fail_ci_if_error: true
          verbose: true
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results-${{ matrix.test-type }}
          path: |
            coverage/
            test-results/
            reports/

  # Stage 4: End-to-End Testing
  e2e-tests:
    runs-on: ubuntu-latest
    needs: [build, code-quality]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Restore build cache
        uses: actions/cache@v3
        with:
          path: |
            dist/
            .next/
            build/
          key: build-${{ github.sha }}
      
      - name: Start application
        run: |
          pnpm run start &
          npx wait-on http://localhost:3000 --timeout 60000
        env:
          NODE_ENV: production
      
      - name: Run E2E tests
        run: pnpm run test:e2e
        env:
          CI: true
          PLAYWRIGHT_JUNIT_OUTPUT_NAME: e2e-results.xml
      
      - name: Upload E2E test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: e2e-test-results
          path: |
            test-results/
            playwright-report/

  # Stage 5: Security Scanning
  security:
    runs-on: ubuntu-latest
    needs: build
    permissions:
      security-events: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript
          queries: security-and-quality
      
      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
      
      - name: Run Semgrep
        uses: semgrep/semgrep-action@v1
        with:
          config: auto
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'frontend-app'
          path: '.'
          format: 'SARIF'
          out: 'dependency-check-report'
      
      - name: Upload dependency check results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: dependency-check-report/dependency-check-report.sarif

  # Stage 6: Performance Testing
  performance:
    runs-on: ubuntu-latest
    needs: [build, code-quality]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Restore build cache
        uses: actions/cache@v3
        with:
          path: |
            dist/
            .next/
            build/
          key: build-${{ github.sha }}
      
      - name: Start application
        run: |
          pnpm run start &
          npx wait-on http://localhost:3000 --timeout 60000
      
      - name: Run Lighthouse CI
        run: pnpm run lighthouse:ci
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
      
      - name: Bundle size analysis
        run: |
          pnpm run analyze:bundle
          npx bundlesize
      
      - name: Upload performance reports
        uses: actions/upload-artifact@v4
        with:
          name: performance-reports
          path: |
            lighthouse-report.html
            bundle-analyzer-report.html

  # Stage 7: Build and Push Container
  containerize:
    runs-on: ubuntu-latest
    needs: [test, e2e-tests, security, performance]
    if: needs.build.outputs.should-deploy == 'true'
    outputs:
      image: ${{ steps.image.outputs.image }}
      digest: ${{ steps.build.outputs.digest }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha,prefix={{branch}}-
      
      - name: Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            VERSION=${{ needs.build.outputs.version }}
            BUILD_TIME=${{ fromJSON(steps.meta.outputs.json).created }}
            GIT_SHA=${{ github.sha }}
      
      - name: Output image
        id: image
        run: echo "image=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}" >> $GITHUB_OUTPUT

  # Stage 8: Deploy to Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build, containerize]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.myapp.com
    
    steps:
      - name: Deploy to staging
        uses: ./.github/actions/deploy
        with:
          environment: staging
          image: ${{ needs.containerize.outputs.image }}
          kubeconfig: ${{ secrets.KUBECONFIG_STAGING }}
      
      - name: Run smoke tests
        run: |
          sleep 60  # Wait for deployment to stabilize
          npx wait-on https://staging.myapp.com --timeout 300000
          pnpm run test:smoke --env=staging
      
      - name: Notify team
        uses: 8398a7/action-slack@v3
        if: always()
        with:
          status: ${{ job.status }}
          text: |
            🚀 Staging deployment ${{ job.status }}
            📊 Version: ${{ needs.build.outputs.version }}
            🔗 URL: https://staging.myapp.com
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Stage 9: Production Deployment (Manual Approval)
  deploy-production:
    runs-on: ubuntu-latest
    needs: [build, containerize, deploy-staging]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - name: Deploy to production
        uses: ./.github/actions/deploy
        with:
          environment: production
          image: ${{ needs.containerize.outputs.image }}
          kubeconfig: ${{ secrets.KUBECONFIG_PRODUCTION }}
          feature-flags: ${{ secrets.PRODUCTION_FEATURE_FLAGS }}
      
      - name: Run production smoke tests
        run: |
          sleep 120  # More time for production deployment
          npx wait-on https://myapp.com --timeout 300000
          pnpm run test:smoke --env=production
      
      - name: Create GitHub release
        uses: actions/create-release@v1
        if: success()
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ needs.build.outputs.version }}
          release_name: Release v${{ needs.build.outputs.version }}
          body: |
            🎉 Production deployment successful!
            
            **Changes:**
            ${{ github.event.head_commit.message }}
            
            **Image:** ${{ needs.containerize.outputs.image }}
            **Digest:** ${{ needs.containerize.outputs.digest }}
      
      - name: Notify stakeholders
        uses: 8398a7/action-slack@v3
        if: always()
        with:
          status: ${{ job.status }}
          text: |
            🎉 Production deployment ${{ job.status }}!
            📊 Version: v${{ needs.build.outputs.version }}
            🔗 URL: https://myapp.com
            📝 Changelog: https://github.com/${{ github.repository }}/releases/tag/v${{ needs.build.outputs.version }}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_PRODUCTION }}

  # Stage 10: Post-Deployment Monitoring
  post-deployment:
    runs-on: ubuntu-latest
    needs: [deploy-production]
    if: always() && needs.deploy-production.result == 'success'
    
    steps:
      - name: Monitor deployment health
        run: |
          # Custom monitoring script
          node scripts/monitor-deployment.js
        env:
          MONITORING_WEBHOOK: ${{ secrets.MONITORING_WEBHOOK }}
          SENTRY_ORG: ${{ secrets.SENTRY_ORG }}
          SENTRY_PROJECT: ${{ secrets.SENTRY_PROJECT }}
          SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
      
      - name: Update status page
        uses: ./.github/actions/update-status-page
        with:
          status: operational
          version: ${{ needs.build.outputs.version }}
          api-key: ${{ secrets.STATUS_PAGE_API_KEY }}
```

## 🐳 Containerization Best Practices

### Multi-Stage Dockerfile

```dockerfile
# Stage 1: Build environment
FROM node:18-alpine AS builder

# Install pnpm
RUN npm install -g pnpm

# Set working directory
WORKDIR /app

# Copy package files
COPY package.json pnpm-lock.yaml ./

# Install dependencies
RUN pnpm install --frozen-lockfile

# Copy source code
COPY . .

# Build arguments
ARG VERSION=latest
ARG BUILD_TIME
ARG GIT_SHA

# Set environment variables for build
ENV NODE_ENV=production
ENV VITE_VERSION=$VERSION
ENV VITE_BUILD_TIME=$BUILD_TIME
ENV VITE_GIT_SHA=$GIT_SHA

# Build application
RUN pnpm run build

# Stage 2: Production environment
FROM nginx:alpine AS production

# Install security updates
RUN apk update && apk upgrade && apk add --no-cache \
    curl \
    ca-certificates \
    && rm -rf /var/cache/apk/*

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-security-headers.conf /etc/nginx/conf.d/security-headers.conf

# Copy built application
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy health check script
COPY scripts/health-check.sh /usr/local/bin/health-check.sh
RUN chmod +x /usr/local/bin/health-check.sh

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# Set ownership
RUN chown -R appuser:appgroup /usr/share/nginx/html && \
    chown -R appuser:appgroup /var/cache/nginx && \
    chown -R appuser:appgroup /var/log/nginx && \
    chown -R appuser:appgroup /etc/nginx/conf.d

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD /usr/local/bin/health-check.sh

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### Nginx Configuration for Production

```nginx
# nginx.conf
user appuser;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /tmp/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    
    # Performance optimizations
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;
    
    # Brotli compression (if module available)
    brotli on;
    brotli_comp_level 6;
    brotli_types
        text/plain
        text/css
        application/json
        application/javascript
        text/xml
        application/xml
        application/xml+rss
        text/javascript;
    
    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=static:10m rate=100r/s;
    
    # Security headers
    include /etc/nginx/conf.d/security-headers.conf;
    
    server {
        listen 8080;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;
        
        # Security
        server_tokens off;
        
        # Static assets with long cache
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
            access_log off;
            
            # Rate limiting for static assets
            limit_req zone=static burst=200 nodelay;
        }
        
        # HTML files with short cache
        location ~* \.html$ {
            expires 5m;
            add_header Cache-Control "public, must-revalidate";
        }
        
        # API proxy with rate limiting
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://backend-service;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
        
        # SPA fallback
        location / {
            try_files $uri $uri/ /index.html;
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        # Deny access to hidden files
        location ~ /\. {
            deny all;
            access_log off;
            log_not_found off;
        }
    }
}
```

### Security Headers Configuration

```nginx
# nginx-security-headers.conf
# Security Headers
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

# Content Security Policy
add_header Content-Security-Policy "
    default-src 'self';
    script-src 'self' 'unsafe-inline' https://www.google-analytics.com;
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    img-src 'self' data: https:;
    font-src 'self' https://fonts.gstatic.com;
    connect-src 'self' https://api.myapp.com https://www.google-analytics.com;
    media-src 'self';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    upgrade-insecure-requests;
" always;

# HSTS (HTTP Strict Transport Security)
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
```

## 🎯 Feature Flags and Progressive Deployment

### Feature Flag Implementation

```typescript
// Feature flag service
export interface FeatureFlag {
  key: string;
  enabled: boolean;
  rolloutPercentage: number;
  conditions?: FeatureFlagCondition[];
  variants?: FeatureFlagVariant[];
}

export interface FeatureFlagCondition {
  type: 'user_id' | 'email' | 'country' | 'browser' | 'custom';
  operator: 'equals' | 'contains' | 'in' | 'not_in';
  value: string | string[];
}

export interface FeatureFlagVariant {
  key: string;
  weight: number;
  payload?: Record<string, any>;
}

class FeatureFlagService {
  private flags = new Map<string, FeatureFlag>();
  private userContext: UserContext;
  
  constructor(userContext: UserContext) {
    this.userContext = userContext;
    this.loadFlags();
  }
  
  async loadFlags(): Promise<void> {
    try {
      const response = await fetch('/api/feature-flags', {
        headers: {
          'Authorization': `Bearer ${this.userContext.token}`,
          'User-Agent': navigator.userAgent,
          'X-User-ID': this.userContext.userId,
        }
      });
      
      const flags: FeatureFlag[] = await response.json();
      
      flags.forEach(flag => {
        this.flags.set(flag.key, flag);
      });
      
      // Cache flags locally
      localStorage.setItem('feature-flags', JSON.stringify(flags));
    } catch (error) {
      console.error('Failed to load feature flags:', error);
      // Load from cache if available
      this.loadFromCache();
    }
  }
  
  isEnabled(flagKey: string): boolean {
    const flag = this.flags.get(flagKey);
    if (!flag) return false;
    
    // Check if globally disabled
    if (!flag.enabled) return false;
    
    // Check conditions
    if (flag.conditions && !this.evaluateConditions(flag.conditions)) {
      return false;
    }
    
    // Check rollout percentage
    if (flag.rolloutPercentage < 100) {
      const hash = this.hashUser(this.userContext.userId, flagKey);
      return hash < flag.rolloutPercentage;
    }
    
    return true;
  }
  
  getVariant(flagKey: string): string | null {
    const flag = this.flags.get(flagKey);
    if (!flag || !this.isEnabled(flagKey) || !flag.variants) {
      return null;
    }
    
    const hash = this.hashUser(this.userContext.userId, `${flagKey}-variant`);
    let cumulative = 0;
    
    for (const variant of flag.variants) {
      cumulative += variant.weight;
      if (hash < cumulative) {
        return variant.key;
      }
    }
    
    return flag.variants[0]?.key || null;
  }
  
  private evaluateConditions(conditions: FeatureFlagCondition[]): boolean {
    return conditions.every(condition => {
      const contextValue = this.getContextValue(condition.type);
      
      switch (condition.operator) {
        case 'equals':
          return contextValue === condition.value;
        case 'contains':
          return typeof contextValue === 'string' && 
                 contextValue.includes(condition.value as string);
        case 'in':
          return Array.isArray(condition.value) && 
                 condition.value.includes(contextValue);
        case 'not_in':
          return Array.isArray(condition.value) && 
                 !condition.value.includes(contextValue);
        default:
          return false;
      }
    });
  }
  
  private getContextValue(type: string): string {
    switch (type) {
      case 'user_id':
        return this.userContext.userId;
      case 'email':
        return this.userContext.email;
      case 'country':
        return this.userContext.country;
      case 'browser':
        return navigator.userAgent;
      default:
        return '';
    }
  }
  
  private hashUser(userId: string, salt: string): number {
    const hash = this.simpleHash(`${userId}-${salt}`);
    return (hash % 100) + 1; // 1-100
  }
  
  private simpleHash(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32bit integer
    }
    return Math.abs(hash);
  }
}

// React hook for feature flags
export const useFeatureFlag = (flagKey: string) => {
  const featureFlagService = useFeatureFlagService();
  const [isEnabled, setIsEnabled] = useState(false);
  const [variant, setVariant] = useState<string | null>(null);
  
  useEffect(() => {
    const enabled = featureFlagService.isEnabled(flagKey);
    const flagVariant = featureFlagService.getVariant(flagKey);
    
    setIsEnabled(enabled);
    setVariant(flagVariant);
    
    // Track feature flag evaluation
    analytics.track('feature_flag_evaluated', {
      flag_key: flagKey,
      enabled,
      variant: flagVariant,
      user_id: featureFlagService.userContext.userId
    });
  }, [flagKey, featureFlagService]);
  
  return { isEnabled, variant };
};

// Feature flag component wrapper
export const FeatureFlag: React.FC<{
  flag: string;
  children: React.ReactNode;
  fallback?: React.ReactNode;
}> = ({ flag, children, fallback = null }) => {
  const { isEnabled } = useFeatureFlag(flag);
  
  return isEnabled ? <>{children}</> : <>{fallback}</>;
};

// Usage examples
const ProductSearch: React.FC = () => {
  const { isEnabled: advancedSearchEnabled } = useFeatureFlag('advanced-search');
  const { variant: searchAlgorithm } = useFeatureFlag('search-algorithm');
  
  return (
    <div>
      {advancedSearchEnabled ? (
        <AdvancedSearchComponent algorithm={searchAlgorithm} />
      ) : (
        <BasicSearchComponent />
      )}
      
      <FeatureFlag flag="voice-search" fallback={<TextSearchOnly />}>
        <VoiceSearchButton />
      </FeatureFlag>
    </div>
  );
};
```

### Blue-Green Deployment Strategy

```yaml
# Kubernetes blue-green deployment
apiVersion: v1
kind: ConfigMap
metadata:
  name: deployment-config
data:
  DEPLOYMENT_STRATEGY: "blue-green"
  HEALTH_CHECK_PATH: "/health"
  READINESS_CHECK_PATH: "/ready"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-blue
  labels:
    app: frontend
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
      version: blue
  template:
    metadata:
      labels:
        app: frontend
        version: blue
    spec:
      containers:
      - name: frontend
        image: myregistry/frontend:latest
        ports:
        - containerPort: 8080
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: VERSION
          value: "blue"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-blue-service
spec:
  selector:
    app: frontend
    version: blue
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP

---
# Green deployment (identical to blue but with different version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-green
  labels:
    app: frontend
    version: green
spec:
  replicas: 0  # Initially scaled to 0
  selector:
    matchLabels:
      app: frontend
      version: green
  template:
    metadata:
      labels:
        app: frontend
        version: green
    spec:
      containers:
      - name: frontend
        image: myregistry/frontend:new-version
        ports:
        - containerPort: 8080
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: VERSION
          value: "green"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
# Main service that switches between blue and green
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
    version: blue  # This gets updated during deployment
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

### Deployment Automation Script

```bash
#!/bin/bash
# Blue-Green deployment script

set -e

NAMESPACE="production"
APP="frontend"
NEW_IMAGE="$1"
CURRENT_VERSION=$(kubectl get service frontend-service -o jsonpath='{.spec.selector.version}')
NEW_VERSION=""

# Determine new version
if [ "$CURRENT_VERSION" = "blue" ]; then
    NEW_VERSION="green"
else
    NEW_VERSION="blue"
fi

echo "🚀 Starting blue-green deployment"
echo "📊 Current version: $CURRENT_VERSION"
echo "🎯 Target version: $NEW_VERSION"
echo "🖼️ New image: $NEW_IMAGE"

# Update the deployment with new image
echo "📦 Updating $NEW_VERSION deployment with new image..."
kubectl set image deployment/frontend-$NEW_VERSION frontend=$NEW_IMAGE -n $NAMESPACE

# Scale up the new version
echo "⬆️ Scaling up $NEW_VERSION deployment..."
kubectl scale deployment frontend-$NEW_VERSION --replicas=3 -n $NAMESPACE

# Wait for new version to be ready
echo "⏳ Waiting for $NEW_VERSION deployment to be ready..."
kubectl rollout status deployment/frontend-$NEW_VERSION -n $NAMESPACE --timeout=300s

# Health check
echo "🏥 Performing health checks..."
for i in {1..30}; do
    if kubectl exec -n $NAMESPACE deployment/frontend-$NEW_VERSION -- curl -f http://localhost:8080/health > /dev/null 2>&1; then
        echo "✅ Health check passed"
        break
    fi
    if [ $i -eq 30 ]; then
        echo "❌ Health check failed after 30 attempts"
        exit 1
    fi
    echo "⏳ Health check attempt $i/30..."
    sleep 10
done

# Run smoke tests
echo "🧪 Running smoke tests..."
kubectl run smoke-test --image=myregistry/smoke-tests:latest --rm -i --restart=Never \
  --env="TARGET_URL=http://frontend-$NEW_VERSION-service" -- npm run smoke-test

if [ $? -ne 0 ]; then
    echo "❌ Smoke tests failed, rolling back..."
    kubectl scale deployment frontend-$NEW_VERSION --replicas=0 -n $NAMESPACE
    exit 1
fi

# Switch traffic to new version
echo "🔄 Switching traffic to $NEW_VERSION..."
kubectl patch service frontend-service -p '{"spec":{"selector":{"version":"'$NEW_VERSION'"}}}' -n $NAMESPACE

# Wait a bit for traffic to switch
sleep 30

# Final verification
echo "🔍 Final verification..."
for i in {1..10}; do
    if curl -f http://frontend-service/health > /dev/null 2>&1; then
        echo "✅ Traffic switch successful"
        break
    fi
    if [ $i -eq 10 ]; then
        echo "❌ Traffic switch verification failed"
        # Rollback
        kubectl patch service frontend-service -p '{"spec":{"selector":{"version":"'$CURRENT_VERSION'"}}}' -n $NAMESPACE
        exit 1
    fi
    sleep 5
done

# Scale down old version
echo "⬇️ Scaling down $CURRENT_VERSION deployment..."
kubectl scale deployment frontend-$CURRENT_VERSION --replicas=0 -n $NAMESPACE

echo "🎉 Blue-green deployment completed successfully!"
echo "📊 Active version: $NEW_VERSION"
echo "🔗 Application URL: http://frontend-service"

# Notify team
curl -X POST $SLACK_WEBHOOK -H 'Content-type: application/json' \
  --data "{\"text\":\"🎉 Production deployment successful!\n📊 Version: $NEW_VERSION\n🖼️ Image: $NEW_IMAGE\"}"
```

## 📊 Environment Management

### Environment Configuration Strategy

```typescript
// Environment configuration management
interface Environment {
  name: string;
  apiUrl: string;
  features: FeatureFlags;
  monitoring: MonitoringConfig;
  security: SecurityConfig;
}

// Environment-specific configurations
export const environments: Record<string, Environment> = {
  development: {
    name: 'development',
    apiUrl: 'http://localhost:8000',
    features: {
      debugMode: true,
      mockData: true,
      advancedSearch: true,
      betaFeatures: true
    },
    monitoring: {
      enableAnalytics: false,
      enableErrorReporting: false,
      logLevel: 'debug'
    },
    security: {
      enableCSP: false,
      allowedOrigins: ['http://localhost:3000'],
      enableHTTPS: false
    }
  },
  
  staging: {
    name: 'staging',
    apiUrl: 'https://api-staging.myapp.com',
    features: {
      debugMode: true,
      mockData: false,
      advancedSearch: true,
      betaFeatures: true
    },
    monitoring: {
      enableAnalytics: true,
      enableErrorReporting: true,
      logLevel: 'info'
    },
    security: {
      enableCSP: true,
      allowedOrigins: ['https://staging.myapp.com'],
      enableHTTPS: true
    }
  },
  
  production: {
    name: 'production',
    apiUrl: 'https://api.myapp.com',
    features: {
      debugMode: false,
      mockData: false,
      advancedSearch: true,
      betaFeatures: false
    },
    monitoring: {
      enableAnalytics: true,
      enableErrorReporting: true,
      logLevel: 'warn'
    },
    security: {
      enableCSP: true,
      allowedOrigins: ['https://myapp.com'],
      enableHTTPS: true
    }
  }
};

// Configuration loader with validation
export class ConfigurationService {
  private config: Environment;
  
  constructor() {
    this.config = this.loadConfiguration();
    this.validateConfiguration();
  }
  
  private loadConfiguration(): Environment {
    const envName = process.env.NODE_ENV || 'development';
    const config = environments[envName];
    
    if (!config) {
      throw new Error(`Unknown environment: ${envName}`);
    }
    
    // Override with environment variables
    return {
      ...config,
      apiUrl: process.env.VITE_API_URL || config.apiUrl,
      features: {
        ...config.features,
        ...this.parseFeatureFlags(process.env.VITE_FEATURE_FLAGS)
      }
    };
  }
  
  private parseFeatureFlags(flagsString?: string): Partial<FeatureFlags> {
    if (!flagsString) return {};
    
    try {
      return JSON.parse(flagsString);
    } catch {
      console.warn('Invalid feature flags format');
      return {};
    }
  }
  
  private validateConfiguration(): void {
    const errors: string[] = [];
    
    if (!this.config.apiUrl) {
      errors.push('API URL is required');
    }
    
    if (!this.config.apiUrl.startsWith('http')) {
      errors.push('API URL must be a valid HTTP/HTTPS URL');
    }
    
    if (this.config.name === 'production' && !this.config.security.enableHTTPS) {
      errors.push('HTTPS must be enabled in production');
    }
    
    if (errors.length > 0) {
      throw new Error(`Configuration validation failed: ${errors.join(', ')}`);
    }
  }
  
  get(): Environment {
    return this.config;
  }
  
  isFeatureEnabled(feature: keyof FeatureFlags): boolean {
    return this.config.features[feature] || false;
  }
  
  getApiUrl(): string {
    return this.config.apiUrl;
  }
}
```

### Docker Compose for Local Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - VITE_API_URL=http://localhost:8000
      - VITE_FEATURE_FLAGS={"debugMode":true,"mockData":true}
    depends_on:
      - backend
      - redis
    networks:
      - app-network

  backend:
    image: myregistry/backend:latest
    ports:
      - "8000:8000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://dev:dev@postgres:5432/myapp_dev
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=myapp_dev
      - POSTGRES_USER=dev
      - POSTGRES_PASSWORD=dev
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

  monitoring:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:
  grafana_data:

networks:
  app-network:
    driver: bridge
```

## 📝 Exercise: Complete CI/CD Setup

Build a production-ready CI/CD pipeline for your project:

### Step 1: Pipeline Design
- Map out your deployment stages and quality gates
- Define environment promotion strategy
- Plan rollback and monitoring procedures
- Design feature flag strategy

### Step 2: Implementation
- Create comprehensive GitHub Actions workflow
- Set up containerization with security best practices
- Implement blue-green or canary deployment
- Configure environment-specific settings

### Step 3: Quality Gates
- Add automated testing at all stages
- Implement security scanning and compliance checks
- Set up performance regression testing
- Create deployment approval processes

### Step 4: Monitoring & Observability
- Set up deployment health monitoring
- Create alerting for deployment failures
- Implement rollback automation triggers
- Build deployment dashboards

**Deliverable:** A complete CI/CD pipeline with automated testing, security scanning, progressive deployment, and comprehensive monitoring.

---

**Next:** [Module 13: Monitoring & Observability](../13-monitoring-observability/) - Master error tracking, performance monitoring, and production observability for frontend applications.