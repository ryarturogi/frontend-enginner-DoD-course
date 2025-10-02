# Module 12: Enterprise DevOps & Continuous Delivery

## 🎯 Learning Objectives

By the end of this module, you will:
- Architect enterprise-grade CI/CD pipelines with zero-downtime deployments and automated rollbacks
- Implement advanced deployment strategies including canary, blue-green, and ring deployments
- Master GitOps workflows with infrastructure as code and declarative deployments
- Build comprehensive security-first pipelines with supply chain protection and compliance automation
- Design intelligent quality gates with ML-powered failure prediction and automated remediation
- Implement chaos engineering and resilience testing in deployment pipelines
- Create self-healing infrastructure with auto-scaling and intelligent monitoring
- Master container orchestration with Kubernetes operators and service mesh integration

## 🏗️ Enterprise DevOps Architecture

### Modern DevOps Platform Overview

```mermaid
graph TB
    A[Developer] --> B[Git Repository]
    B --> C[GitOps Controller]
    C --> D[CI/CD Pipeline]
    D --> E[Security Gates]
    E --> F[Quality Gates]
    F --> G[Artifact Registry]
    G --> H[Deployment Engine]
    H --> I[Service Mesh]
    I --> J[Observability Stack]
    J --> K[Alert Manager]
    K --> L[Auto-Remediation]
    L --> M[Rollback Engine]
    
    N[Infrastructure as Code] --> O[Terraform Cloud]
    O --> P[Kubernetes Clusters]
    P --> I
    
    Q[Feature Flags] --> R[Progressive Delivery]
    R --> H
    
    S[Chaos Engineering] --> T[Resilience Testing]
    T --> P
```

### Enterprise DevOps Principles

```typescript
// Enterprise DevOps Platform Architecture
interface DevOpsPlatform {
  // Infrastructure Layer
  infrastructure: {
    multiCloud: 'AWS + Azure + GCP for resilience';
    kubernetes: 'Multi-cluster with service mesh';
    networking: 'Zero-trust network with mTLS';
    storage: 'Distributed with automated backup';
  };
  
  // Security Layer
  security: {
    shiftLeft: 'Security built into every stage';
    zeroTrust: 'Never trust, always verify';
    supplyChain: 'Signed artifacts and SBOM';
    compliance: 'Automated SOC2, ISO27001, PCI-DSS';
  };
  
  // Deployment Layer
  deployment: {
    gitOps: 'Declarative infrastructure and apps';
    progressive: 'Canary, blue-green, ring deployments';
    automation: 'Fully automated with manual overrides';
    rollback: 'Instant rollback with state preservation';
  };
  
  // Observability Layer
  observability: {
    metrics: 'Real-time with predictive analytics';
    logging: 'Centralized with intelligent search';
    tracing: 'Distributed with performance insights';
    alerting: 'Context-aware with auto-remediation';
  };
  
  // Quality Layer
  quality: {
    testing: 'Comprehensive with AI-powered generation';
    performance: 'Continuous with regression detection';
    security: 'Automated with zero-day protection';
    compliance: 'Continuous with audit trails';
  };
}
```

### GitOps Workflow Implementation

```typescript
// GitOps Configuration Management
interface GitOpsConfig {
  repositories: {
    application: 'Source code and application config';
    infrastructure: 'Terraform and K8s manifests';
    configuration: 'Environment-specific settings';
    policies: 'Security and compliance policies';
  };
  
  controllers: {
    argoCD: 'Application deployment automation';
    flux: 'Infrastructure synchronization';
    tekton: 'Pipeline orchestration';
    crossplane: 'Cloud resource management';
  };
  
  sync: {
    interval: 'Every 30 seconds for critical apps';
    strategy: 'Rolling updates with health checks';
    validation: 'Pre-deployment validation and testing';
    rollback: 'Automatic on failure detection';
  };
}

// ArgoCD Application Definition
export const applicationManifest = {
  apiVersion: 'argoproj.io/v1alpha1',
  kind: 'Application',
  metadata: {
    name: 'frontend-app',
    namespace: 'argocd',
    labels: {
      'app.kubernetes.io/name': 'frontend-app',
      'app.kubernetes.io/component': 'web',
      'app.kubernetes.io/part-of': 'ecommerce-platform'
    },
    annotations: {
      'argocd.argoproj.io/sync-wave': '2',
      'argocd.argoproj.io/hook': 'PreSync',
      'notifications.argoproj.io/subscribe.on-deployed.slack': 'deployments',
      'notifications.argoproj.io/subscribe.on-health-degraded.slack': 'alerts'
    }
  },
  spec: {
    project: 'default',
    source: {
      repoURL: 'https://github.com/company/frontend-app-config',
      targetRevision: 'HEAD',
      path: 'manifests/production',
      helm: {
        valueFiles: ['values.yaml', 'values-production.yaml'],
        parameters: [
          {
            name: 'image.tag',
            value: '$ARGOCD_APP_REVISION'
          },
          {
            name: 'replicaCount',
            value: '5'
          }
        ]
      }
    },
    destination: {
      server: 'https://kubernetes.default.svc',
      namespace: 'frontend-production'
    },
    syncPolicy: {
      automated: {
        prune: true,
        selfHeal: true,
        allowEmpty: false
      },
      syncOptions: [
        'CreateNamespace=true',
        'PrunePropagationPolicy=foreground',
        'PruneLast=true',
        'RespectIgnoreDifferences=true'
      ],
      retry: {
        limit: 3,
        backoff: {
          duration: '5s',
          factor: 2,
          maxDuration: '3m'
        }
      }
    },
    ignoreDifferences: [
      {
        group: 'apps',
        kind: 'Deployment',
        jsonPointers: ['/spec/replicas']
      }
    ]
  }
};
```

### Infrastructure as Code with Terraform

```hcl
# terraform/environments/production/main.tf
# Enterprise-grade Kubernetes cluster with GitOps

terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.0"
    }
    argocd = {
      source  = "oboukili/argocd"
      version = "~> 6.0"
    }
  }
  
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "frontend/production/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Multi-AZ EKS cluster with managed node groups
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = "frontend-production"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  # Cluster endpoint configuration
  cluster_endpoint_private_access = true
  cluster_endpoint_public_access  = true
  cluster_endpoint_public_access_cidrs = ["0.0.0.0/0"]
  
  # OIDC Identity provider
  enable_irsa = true
  
  # Encryption
  cluster_encryption_config = [
    {
      provider_key_arn = aws_kms_key.eks.arn
      resources        = ["secrets"]
    }
  ]
  
  # Logging
  cluster_enabled_log_types = [
    "api", "audit", "authenticator", "controllerManager", "scheduler"
  ]
  
  # Managed node groups
  eks_managed_node_groups = {
    main = {
      name = "main-nodes"
      
      instance_types = ["c5.xlarge"]
      capacity_type  = "ON_DEMAND"
      
      min_size     = 3
      max_size     = 20
      desired_size = 5
      
      # Launch template
      create_launch_template = true
      launch_template_name   = "main-nodes"
      
      # Node group configuration
      ami_type = "AL2_x86_64"
      disk_size = 100
      disk_type = "gp3"
      
      # Labels and taints
      labels = {
        Environment = "production"
        NodeGroup   = "main"
      }
      
      taints = {
        dedicated = {
          key    = "CriticalAddonsOnly"
          value  = "true"
          effect = "NO_SCHEDULE"
        }
      }
      
      # Auto Scaling Group tags
      asg_tags = {
        AsgTag = "main-nodes"
      }
    }
    
    spot = {
      name = "spot-nodes"
      
      instance_types = ["c5.large", "c5.xlarge", "c5.2xlarge"]
      capacity_type  = "SPOT"
      
      min_size     = 2
      max_size     = 50
      desired_size = 10
      
      labels = {
        Environment = "production"
        NodeGroup   = "spot"
        NodeType    = "spot"
      }
      
      taints = {
        spot = {
          key    = "spot"
          value  = "true"
          effect = "NO_SCHEDULE"
        }
      }
    }
  }
  
  # aws-auth configmap
  manage_aws_auth_configmap = true
  
  aws_auth_roles = [
    {
      rolearn  = aws_iam_role.developer.arn
      username = "developer"
      groups   = ["system:masters"]
    },
  ]
  
  tags = {
    Environment = "production"
    Project     = "frontend"
    ManagedBy   = "terraform"
  }
}

# Install ArgoCD using Helm
resource "helm_release" "argocd" {
  name       = "argocd"
  repository = "https://argoproj.github.io/argo-helm"
  chart      = "argo-cd"
  namespace  = "argocd"
  version    = "5.46.7"
  
  create_namespace = true
  
  values = [templatefile("${path.module}/argocd-values.yaml", {
    domain = "argocd.${var.domain_name}"
    certificate_arn = aws_acm_certificate.argocd.arn
  })]
  
  depends_on = [
    module.eks,
    kubernetes_namespace.argocd
  ]
}

# Install Istio service mesh
resource "helm_release" "istio_base" {
  name       = "istio-base"
  repository = "https://istio-release.storage.googleapis.com/charts"
  chart      = "base"
  namespace  = "istio-system"
  version    = "1.19.3"
  
  create_namespace = true
  
  depends_on = [module.eks]
}

resource "helm_release" "istiod" {
  name       = "istiod"
  repository = "https://istio-release.storage.googleapis.com/charts"
  chart      = "istiod"
  namespace  = "istio-system"
  version    = "1.19.3"
  
  values = [
    yamlencode({
      pilot = {
        traceSampling = 1.0
        resources = {
          requests = {
            cpu    = "100m"
            memory = "128Mi"
          }
          limits = {
            cpu    = "500m"
            memory = "512Mi"
          }
        }
      }
    })
  ]
  
  depends_on = [helm_release.istio_base]
}

# Monitoring stack with Prometheus and Grafana
resource "helm_release" "kube_prometheus_stack" {
  name       = "kube-prometheus-stack"
  repository = "https://prometheus-community.github.io/helm-charts"
  chart      = "kube-prometheus-stack"
  namespace  = "monitoring"
  version    = "51.2.0"
  
  create_namespace = true
  
  values = [templatefile("${path.module}/monitoring-values.yaml", {
    storage_class = "gp3"
    grafana_domain = "grafana.${var.domain_name}"
    prometheus_domain = "prometheus.${var.domain_name}"
  })]
  
  depends_on = [module.eks]
}

# Cluster autoscaler
resource "helm_release" "cluster_autoscaler" {
  name       = "cluster-autoscaler"
  repository = "https://kubernetes.github.io/autoscaler"
  chart      = "cluster-autoscaler"
  namespace  = "kube-system"
  version    = "9.29.0"
  
  values = [
    yamlencode({
      autoDiscovery = {
        clusterName = module.eks.cluster_name
      }
      awsRegion = var.aws_region
      rbac = {
        serviceAccount = {
          annotations = {
            "eks.amazonaws.com/role-arn" = aws_iam_role.cluster_autoscaler.arn
          }
        }
      }
    })
  ]
  
  depends_on = [module.eks]
}
```

### Advanced Security Pipeline

```yaml
# .github/workflows/security-first-pipeline.yml
name: Security-First CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  COSIGN_EXPERIMENTAL: 1

permissions:
  contents: read
  security-events: write
  packages: write
  id-token: write

jobs:
  # Supply Chain Security
  supply-chain-security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout with full history
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Verify commit signatures
        run: |
          git log --show-signature --oneline -10
          
      - name: SLSA Provenance Generation
        uses: slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@v1.9.0
        with:
          base64-subjects: "${{ steps.hash.outputs.base64-subjects }}"
          
      - name: Install Cosign
        uses: sigstore/cosign-installer@v3
        
      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          format: spdx-json
          output-file: sbom.spdx.json
          
      - name: Sign SBOM
        run: |
          cosign sign-blob --bundle cosign.bundle sbom.spdx.json
          
      - name: Upload SBOM
        uses: actions/upload-artifact@v4
        with:
          name: sbom
          path: |
            sbom.spdx.json
            cosign.bundle

  # Advanced Code Analysis
  advanced-security:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        scanner: [codeql, semgrep, snyk, sonarcloud]
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: CodeQL Analysis
        if: matrix.scanner == 'codeql'
        uses: github/codeql-action/init@v3
        with:
          languages: javascript
          queries: security-extended,security-and-quality
          
      - name: Perform CodeQL Analysis
        if: matrix.scanner == 'codeql'
        uses: github/codeql-action/analyze@v3
        
      - name: Semgrep Scan
        if: matrix.scanner == 'semgrep'
        uses: semgrep/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
            p/react
            p/typescript
            
      - name: Snyk Security Scan
        if: matrix.scanner == 'snyk'
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --fail-on=upgradable
          
      - name: SonarCloud Scan
        if: matrix.scanner == 'sonarcloud'
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # Container Security
  container-security:
    runs-on: ubuntu-latest
    needs: [supply-chain-security]
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Build container for scanning
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: scan-image:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
          
      - name: Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: scan-image:latest
          format: sarif
          output: trivy-results.sarif
          severity: 'CRITICAL,HIGH,MEDIUM'
          
      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif
          
      - name: Docker Scout vulnerability scan
        uses: docker/scout-action@v1
        with:
          command: cves
          image: scan-image:latest
          sarif-file: scout-results.sarif
          summary: true
          
      - name: Dockerfile linting
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
          format: sarif
          output-file: hadolint-results.sarif
          
      - name: Upload Dockerfile lint results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: hadolint-results.sarif

  # Performance & Quality Gates
  intelligent-quality-gates:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Build application
        run: pnpm run build
        
      - name: Bundle analysis with AI insights
        run: |
          pnpm run analyze:bundle
          node scripts/ai-bundle-analysis.js
          
      - name: Lighthouse CI with ML predictions
        run: |
          pnpm run lighthouse:ci
          node scripts/performance-ml-analysis.js
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          
      - name: Quality gate evaluation
        run: node scripts/intelligent-quality-gate.js
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          
      - name: Predictive failure analysis
        uses: ./.github/actions/ml-failure-prediction
        with:
          historical-data: ${{ secrets.PIPELINE_ANALYTICS }}
          current-metrics: ./quality-metrics.json

  # Multi-Architecture Container Build
  secure-container-build:
    runs-on: ubuntu-latest
    needs: [advanced-security, container-security, intelligent-quality-gates]
    outputs:
      image: ${{ steps.image.outputs.image }}
      digest: ${{ steps.build.outputs.digest }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Install Cosign
        uses: sigstore/cosign-installer@v3
        
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
            
      - name: Build and push multi-arch image
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
          provenance: true
          sbom: true
          
      - name: Sign container image
        run: |
          cosign sign --yes ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }}
          
      - name: Verify signature
        run: |
          cosign verify ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }} \
            --certificate-identity https://github.com/${{ github.repository }}/.github/workflows/security-first-pipeline.yml@refs/heads/${{ github.ref_name }} \
            --certificate-oidc-issuer https://token.actions.githubusercontent.com
            
      - name: Generate attestation
        run: |
          cosign attest --yes --predicate sbom.spdx.json \
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }}
```

## 🔄 Advanced CI/CD Pipeline Architecture

### Enterprise Pipeline Overview

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

## 🌊 Chaos Engineering & Resilience Testing

### Chaos Engineering Implementation

```typescript
// Chaos Engineering Framework for Frontend Applications
interface ChaosExperiment {
  id: string;
  name: string;
  description: string;
  target: ChaosTarget;
  faultType: FaultType;
  duration: number;
  impact: ImpactAssessment;
  rollbackTriggers: RollbackTrigger[];
  metrics: MetricDefinition[];
}

enum FaultType {
  NETWORK_LATENCY = 'network-latency',
  NETWORK_PARTITION = 'network-partition',
  SERVICE_UNAVAILABLE = 'service-unavailable',
  MEMORY_PRESSURE = 'memory-pressure',
  CPU_PRESSURE = 'cpu-pressure',
  DISK_PRESSURE = 'disk-pressure',
  DNS_FAILURE = 'dns-failure',
  SSL_CERTIFICATE_EXPIRY = 'ssl-certificate-expiry'
}

// Chaos Experiments for Frontend
export class FrontendChaosExperiments {
  private experiments: Map<string, ChaosExperiment> = new Map();
  
  // API Latency Chaos
  createAPILatencyExperiment(): ChaosExperiment {
    return {
      id: 'api-latency-chaos',
      name: 'API Response Latency Injection',
      description: 'Inject artificial latency into API responses to test timeout handling',
      target: {
        type: 'service',
        selector: 'app=frontend-api',
        namespace: 'production'
      },
      faultType: FaultType.NETWORK_LATENCY,
      duration: 300, // 5 minutes
      impact: {
        severity: 'medium',
        expectedUserImpact: 'Slower page loads, potential timeouts',
        businessImpact: 'May affect conversion rates'
      },
      rollbackTriggers: [
        {
          type: 'metric',
          condition: 'error_rate > 5%',
          action: 'immediate_rollback'
        },
        {
          type: 'metric',
          condition: 'response_time_p95 > 10s',
          action: 'immediate_rollback'
        }
      ],
      metrics: [
        { name: 'error_rate', threshold: 5, unit: '%' },
        { name: 'response_time_p95', threshold: 10000, unit: 'ms' },
        { name: 'user_satisfaction_score', threshold: 7, unit: 'score' }
      ]
    };
  }
  
  // Memory Pressure Chaos
  createMemoryPressureExperiment(): ChaosExperiment {
    return {
      id: 'memory-pressure-chaos',
      name: 'Frontend Container Memory Pressure',
      description: 'Consume memory in frontend containers to test graceful degradation',
      target: {
        type: 'deployment',
        selector: 'app=frontend-web',
        namespace: 'production'
      },
      faultType: FaultType.MEMORY_PRESSURE,
      duration: 180, // 3 minutes
      impact: {
        severity: 'high',
        expectedUserImpact: 'Slower interactions, potential crashes',
        businessImpact: 'May cause user abandonment'
      },
      rollbackTriggers: [
        {
          type: 'metric',
          condition: 'memory_usage > 90%',
          action: 'immediate_rollback'
        },
        {
          type: 'health_check',
          condition: 'pod_restart_count > 2',
          action: 'immediate_rollback'
        }
      ],
      metrics: [
        { name: 'memory_usage', threshold: 85, unit: '%' },
        { name: 'cpu_usage', threshold: 80, unit: '%' },
        { name: 'active_connections', threshold: 1000, unit: 'count' }
      ]
    };
  }
}

// Chaos Mesh Configuration for Kubernetes
export const chaosExperimentManifest = {
  apiVersion: 'chaos-mesh.org/v1alpha1',
  kind: 'NetworkChaos',
  metadata: {
    name: 'frontend-api-latency',
    namespace: 'chaos-engineering',
    labels: {
      'experiment-type': 'network-latency',
      'target-service': 'frontend-api',
      'severity': 'medium'
    }
  },
  spec: {
    action: 'delay',
    mode: 'all',
    selector: {
      namespaces: ['production'],
      labelSelectors: {
        'app': 'frontend-api'
      }
    },
    delay: {
      latency: '2s',
      correlation: '100',
      jitter: '500ms'
    },
    duration: '5m',
    scheduler: {
      cron: '@every 1h' // Run every hour during business hours
    },
    // Safety measures
    preCheck: {
      enabled: true,
      timeout: '30s'
    },
    // Automatic rollback conditions
    conditions: [
      {
        type: 'metric',
        metricName: 'frontend_error_rate',
        operator: 'GreaterThan',
        value: '5',
        duration: '1m'
      }
    ]
  }
};
```

### Progressive Delivery with Argo Rollouts

```yaml
# Canary Deployment with Argo Rollouts
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: frontend-rollout
  namespace: production
  labels:
    app: frontend
    version: v2
spec:
  replicas: 10
  strategy:
    canary:
      # Analysis configuration
      analysis:
        templates:
        - templateName: success-rate
        - templateName: latency-p95
        - templateName: error-rate
        startingStep: 2 # Start analysis after 20% traffic
        args:
        - name: service-name
          value: frontend-service
        - name: namespace
          value: production
      
      # Canary steps
      steps:
      - setWeight: 5    # 5% traffic to canary
      - pause:
          duration: 2m  # Observe for 2 minutes
      
      - setWeight: 10   # 10% traffic to canary
      - pause:
          duration: 5m  # Extended observation
      
      - setWeight: 20   # 20% traffic to canary
      - pause: {}       # Indefinite pause for manual verification
      
      - setWeight: 50   # 50% traffic to canary
      - pause:
          duration: 10m
      
      - setWeight: 100  # Full traffic to canary
      
      # Traffic routing
      trafficRouting:
        istio:
          virtualService:
            name: frontend-vs
            routes:
            - primary
          destinationRule:
            name: frontend-dr
            canarySubsetName: canary
            stableSubsetName: stable
      
      # Anti-affinity for pod distribution
      antiAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          podAntiAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - frontend
              topologyKey: kubernetes.io/hostname
      
      # Automatic rollback configuration
      abortScaleDownDelaySeconds: 30
      scaleDownDelaySeconds: 30
      maxUnavailable: 1
      
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        version: v2
    spec:
      containers:
      - name: frontend
        image: myregistry/frontend:v2.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Environment configuration
        env:
        - name: NODE_ENV
          value: "production"
        - name: LOG_LEVEL
          value: "info"
        - name: METRICS_ENABLED
          value: "true"

---
# Analysis Template for Success Rate
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
  namespace: production
spec:
  args:
  - name: service-name
  - name: namespace
  
  metrics:
  - name: success-rate
    successCondition: result[0] >= 0.95
    failureCondition: result[0] < 0.90
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",code=~"2.."}[5m])) /
          sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))
    interval: 1m
    count: 5
    
  - name: latency-p95
    successCondition: result[0] <= 1000
    failureCondition: result[0] > 2000
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{service="{{args.service-name}}"}[5m])) by (le)
          ) * 1000
    interval: 1m
    count: 5
    
  - name: error-rate
    successCondition: result[0] <= 0.05
    failureCondition: result[0] > 0.10
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc.cluster.local:9090
        query: |
          sum(rate(http_requests_total{service="{{args.service-name}}",code=~"5.."}[5m])) /
          sum(rate(http_requests_total{service="{{args.service-name}}"}[5m]))
    interval: 1m
    count: 5

---
# Ring Deployment for Global Rollout
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: frontend-ring-deployment
  namespace: production
spec:
  replicas: 50
  strategy:
    canary:
      # Ring-based deployment
      steps:
      # Ring 1: Internal employees (1% of traffic)
      - setWeight: 1
        setCanaryScale:
          weight: 10 # 10% of pods for 1% traffic (better observability)
      - pause:
          duration: 1h
      
      # Ring 2: Beta users (5% of traffic)
      - setWeight: 5
        setCanaryScale:
          weight: 20
      - pause:
          duration: 6h
      
      # Ring 3: Regional rollout (25% of traffic)
      - setWeight: 25
        setCanaryScale:
          weight: 40
      - pause:
          duration: 24h
      
      # Ring 4: Full rollout
      - setWeight: 100
      
      # Advanced traffic management
      trafficRouting:
        istio:
          virtualService:
            name: frontend-ring-vs
            routes:
            - primary
          destinationRule:
            name: frontend-ring-dr
            canarySubsetName: canary
            stableSubsetName: stable
      
      # Ring-specific analysis
      analysis:
        templates:
        - templateName: ring-analysis
        args:
        - name: ring-id
          value: "{{metadata.labels.ring-id}}"
        - name: service-name
          value: frontend-service
```

### Intelligent Rollback System

```typescript
// AI-Powered Rollback Decision Engine
export class IntelligentRollbackSystem {
  private metricsCollector: MetricsCollector;
  private mlModel: RollbackPredictionModel;
  private alertManager: AlertManager;
  
  constructor(
    metricsCollector: MetricsCollector,
    mlModel: RollbackPredictionModel,
    alertManager: AlertManager
  ) {
    this.metricsCollector = metricsCollector;
    this.mlModel = mlModel;
    this.alertManager = alertManager;
  }
  
  async evaluateDeploymentHealth(deploymentId: string): Promise<RollbackDecision> {
    // Collect real-time metrics
    const metrics = await this.metricsCollector.getDeploymentMetrics(deploymentId, {
      timeRange: '10m',
      includeComparison: true // Compare with previous stable deployment
    });
    
    // Extract key performance indicators
    const kpis = this.extractKPIs(metrics);
    
    // Get ML prediction
    const prediction = await this.mlModel.predictRollbackNeed(kpis);
    
    // Apply business rules
    const businessRuleViolations = this.checkBusinessRules(kpis);
    
    // Make rollback decision
    const decision = this.makeRollbackDecision(prediction, businessRuleViolations, kpis);
    
    // Log decision with reasoning
    this.logRollbackDecision(deploymentId, decision, kpis, prediction);
    
    return decision;
  }
  
  private extractKPIs(metrics: DeploymentMetrics): KPISnapshot {
    return {
      // Performance KPIs
      errorRate: metrics.errorRate.current,
      errorRateChange: metrics.errorRate.changeFromBaseline,
      
      responseTime: {
        p50: metrics.responseTime.p50,
        p95: metrics.responseTime.p95,
        p99: metrics.responseTime.p99
      },
      responseTimeChange: metrics.responseTime.changeFromBaseline,
      
      // Business KPIs
      conversionRate: metrics.business.conversionRate,
      conversionRateChange: metrics.business.conversionRateChange,
      
      userSatisfactionScore: metrics.business.userSatisfactionScore,
      
      // Infrastructure KPIs
      cpuUsage: metrics.infrastructure.cpuUsage,
      memoryUsage: metrics.infrastructure.memoryUsage,
      
      // Custom metrics
      customMetrics: metrics.custom
    };
  }
  
  private checkBusinessRules(kpis: KPISnapshot): BusinessRuleViolation[] {
    const violations: BusinessRuleViolation[] = [];
    
    // Critical error rate threshold
    if (kpis.errorRate > 5) {
      violations.push({
        type: 'critical',
        rule: 'error-rate-threshold',
        message: `Error rate ${kpis.errorRate}% exceeds critical threshold of 5%`,
        autoRollback: true
      });
    }
    
    // Response time degradation
    if (kpis.responseTimeChange.p95 > 50) { // 50% increase in p95
      violations.push({
        type: 'major',
        rule: 'response-time-degradation',
        message: `P95 response time increased by ${kpis.responseTimeChange.p95}%`,
        autoRollback: true
      });
    }
    
    // Conversion rate impact
    if (kpis.conversionRateChange < -10) { // 10% decrease in conversion
      violations.push({
        type: 'major',
        rule: 'conversion-rate-impact',
        message: `Conversion rate decreased by ${Math.abs(kpis.conversionRateChange)}%`,
        autoRollback: false // Requires human review
      });
    }
    
    // Memory leak detection
    if (kpis.memoryUsage > 85 && this.detectMemoryLeak(kpis)) {
      violations.push({
        type: 'critical',
        rule: 'memory-leak-detection',
        message: 'Potential memory leak detected',
        autoRollback: true
      });
    }
    
    return violations;
  }
  
  private makeRollbackDecision(
    prediction: RollbackPrediction,
    violations: BusinessRuleViolation[],
    kpis: KPISnapshot
  ): RollbackDecision {
    // Auto-rollback for critical violations
    const criticalViolations = violations.filter(v => v.type === 'critical' && v.autoRollback);
    if (criticalViolations.length > 0) {
      return {
        action: 'immediate_rollback',
        confidence: 1.0,
        reason: 'Critical business rule violations detected',
        violations: criticalViolations,
        timeToRollback: '30s'
      };
    }
    
    // ML model high confidence prediction
    if (prediction.confidence > 0.9 && prediction.shouldRollback) {
      return {
        action: 'scheduled_rollback',
        confidence: prediction.confidence,
        reason: `ML model predicts high rollback probability: ${prediction.reasoning}`,
        violations: [],
        timeToRollback: '5m' // Allow time for validation
      };
    }
    
    // Major violations requiring human review
    const majorViolations = violations.filter(v => v.type === 'major');
    if (majorViolations.length > 0) {
      return {
        action: 'alert_human',
        confidence: 0.7,
        reason: 'Major violations detected, human review required',
        violations: majorViolations,
        timeToRollback: null
      };
    }
    
    // Low confidence or mixed signals
    if (prediction.confidence > 0.6 && prediction.shouldRollback) {
      return {
        action: 'monitor_closely',
        confidence: prediction.confidence,
        reason: 'Potential issues detected, monitoring closely',
        violations: [],
        timeToRollback: null,
        monitoringDuration: '30m'
      };
    }
    
    // All good
    return {
      action: 'continue',
      confidence: 1.0 - prediction.confidence,
      reason: 'All metrics within acceptable ranges',
      violations: [],
      timeToRollback: null
    };
  }
  
  private async executeRollback(deploymentId: string, decision: RollbackDecision): Promise<void> {
    const rollbackStartTime = new Date();
    
    try {
      // Update rollout to trigger rollback
      await this.updateRolloutForRollback(deploymentId, decision);
      
      // Monitor rollback progress
      await this.monitorRollbackProgress(deploymentId);
      
      // Verify rollback success
      await this.verifyRollbackSuccess(deploymentId);
      
      // Notify stakeholders
      await this.notifyRollbackSuccess(deploymentId, decision, rollbackStartTime);
      
    } catch (error) {
      await this.handleRollbackFailure(deploymentId, error, rollbackStartTime);
      throw error;
    }
  }
  
  private detectMemoryLeak(kpis: KPISnapshot): boolean {
    // Simple heuristic: memory usage trending upward over time
    // In practice, this would use more sophisticated analysis
    const memoryTrend = kpis.customMetrics?.['memory_usage_trend'];
    return memoryTrend && memoryTrend > 0.1; // 10% increase over monitoring window
  }
}

// Machine Learning Model for Rollback Prediction
export class RollbackPredictionModel {
  private modelEndpoint: string;
  private apiKey: string;
  
  constructor(modelEndpoint: string, apiKey: string) {
    this.modelEndpoint = modelEndpoint;
    this.apiKey = apiKey;
  }
  
  async predictRollbackNeed(kpis: KPISnapshot): Promise<RollbackPrediction> {
    const features = this.extractFeatures(kpis);
    
    const response = await fetch(`${this.modelEndpoint}/predict`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ features })
    });
    
    const prediction = await response.json();
    
    return {
      shouldRollback: prediction.probability > 0.5,
      confidence: prediction.probability,
      reasoning: prediction.explanation,
      riskFactors: prediction.risk_factors,
      recommendedAction: prediction.recommended_action
    };
  }
  
  private extractFeatures(kpis: KPISnapshot): MLFeatures {
    return {
      error_rate: kpis.errorRate,
      error_rate_change: kpis.errorRateChange,
      response_time_p95: kpis.responseTime.p95,
      response_time_change: kpis.responseTimeChange.p95,
      conversion_rate_change: kpis.conversionRateChange,
      cpu_usage: kpis.cpuUsage,
      memory_usage: kpis.memoryUsage,
      user_satisfaction: kpis.userSatisfactionScore,
      
      // Historical context
      deployment_hour: new Date().getHours(),
      deployment_day_of_week: new Date().getDay(),
      traffic_volume: kpis.customMetrics?.['traffic_volume'] || 0,
      
      // Feature flags impact
      feature_flags_changed: kpis.customMetrics?.['feature_flags_changed'] || false,
      
      // Dependencies
      external_service_health: kpis.customMetrics?.['external_service_health'] || 1.0
    };
  }
}
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