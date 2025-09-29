# 18. CI/CD, Docker, Cloud

## Docker

### Khái niệm cơ bản
- **Container**: Môi trường cô lập chứa application và dependencies
- **Image**: Template để tạo container, immutable
- **Dockerfile**: File chứa instruction để build image
- **Registry**: Nơi lưu trữ và phân phối images (Docker Hub, ECR, GCR)

### Docker Commands Cơ Bản
```bash
# Build image từ Dockerfile
docker build -t myapp:1.0 .

# Run container
docker run -d -p 3000:3000 --name myapp-container myapp:1.0

# List containers
docker ps
docker ps -a

# Stop/Start container
docker stop myapp-container
docker start myapp-container

# Remove container/image
docker rm myapp-container
docker rmi myapp:1.0

# View logs
docker logs myapp-container

# Execute command trong container
docker exec -it myapp-container /bin/sh
```

### Dockerfile Best Practices
```dockerfile
# Multi-stage build
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS runtime
WORKDIR /app

# Tạo non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# Copy dependencies
COPY --from=build /app/node_modules ./node_modules
COPY --chown=nextjs:nodejs . .

# Expose port
EXPOSE 3000

# Switch to non-root user
USER nextjs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
```

### Docker Compose
```yaml
version: '3.8'

services:
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./uploads:/app/uploads
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/ssl
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### Docker Networking & Volumes
```bash
# Create custom network
docker network create my-network
docker network ls
docker network inspect my-network

# Create volumes
docker volume create my-volume
docker volume ls
docker volume inspect my-volume

# Mount volume
docker run -v my-volume:/app/data myapp
```

## CI/CD

### Khái niệm CI/CD
- **CI (Continuous Integration)**: Tự động merge, build, test code
- **CD (Continuous Deployment)**: Tự động deploy sau khi CI pass
- **Pipeline**: Chuỗi các bước tự động hóa
- **Artifact**: Sản phẩm build (image, binary, package)

### GitHub Actions
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: password
          POSTGRES_DB: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linting
      run: npm run lint
    
    - name: Run tests
      run: npm run test:coverage
      env:
        DATABASE_URL: postgres://postgres:password@localhost:5432/test
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
    - uses: actions/checkout@v4
    
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
          type=sha,prefix={{branch}}-
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    if: github.ref == 'refs/heads/main'
    needs: [test, build]
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production server"
        # Actual deployment commands here
```

### GitLab CI/CD
```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  image: node:18
  services:
    - postgres:13
  variables:
    POSTGRES_DB: test
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
  before_script:
    - npm ci
  script:
    - npm run lint
    - npm run test:coverage
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE
  only:
    - main
    - develop

deploy_staging:
  stage: deploy
  script:
    - echo "Deploy to staging"
    - docker run -d -p 3000:3000 $DOCKER_IMAGE
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - echo "Deploy to production"
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### Jenkins Pipeline
```groovy
pipeline {
    agent any
    
    environment {
        NODE_VERSION = '18'
        DOCKER_IMAGE = 'myapp'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm run test:coverage'
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'test-results.xml'
                    publishCoverage adapters: [istanbulCoberturaAdapter('coverage/cobertura-coverage.xml')]
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").run("-d -p 3000:3000")
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. Check console output.",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
```

## Cloud Platforms

### AWS Services
```yaml
# ECS Task Definition
{
  "family": "myapp-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "123456789012.dkr.ecr.region.amazonaws.com/myapp:latest",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/myapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        }
      ]
    }
  ]
}
```

### Kubernetes Production Manifests

#### Deployment với Advanced Features
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  namespace: production
  labels:
    app: myapp
    version: v1.0
    tier: backend
  annotations:
    deployment.kubernetes.io/revision: "1"
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: myapp-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        runAsGroup: 1001
        fsGroup: 1001
      containers:
      - name: myapp
        image: myapp:1.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 3000
          protocol: TCP
        env:
        - name: NODE_ENV
          value: "production"
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        envFrom:
        - configMapRef:
            name: myapp-config
        - secretRef:
            name: myapp-secrets
        resources:
          requests:
            memory: "256Mi"
            cpu: "200m"
            ephemeral-storage: "1Gi"
          limits:
            memory: "512Mi"
            cpu: "500m"
            ephemeral-storage: "2Gi"
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
          capabilities:
            drop:
            - ALL
        livenessProbe:
          httpGet:
            path: /health
            port: http
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /startup
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 12
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: app-cache
          mountPath: /app/cache
      volumes:
      - name: tmp
        emptyDir: {}
      - name: app-cache
        emptyDir: {}
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - myapp
              topologyKey: kubernetes.io/hostname
      tolerations:
      - key: "node.kubernetes.io/unreachable"
        operator: "Exists"
        effect: "NoExecute"
        tolerationSeconds: 30

---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  namespace: production
  labels:
    app: myapp
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: http
  sessionAffinity: None

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: production
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/myapp-pod-role

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
  namespace: production
data:
  LOG_LEVEL: "info"
  MAX_CONNECTIONS: "100"
  TIMEOUT: "30s"

---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
  namespace: production
type: Opaque
data:
  DATABASE_URL: <base64-encoded-url>
  API_KEY: <base64-encoded-key>
```

#### Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 30
      - type: Pods
        value: 5
        periodSeconds: 30

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
  namespace: production
spec:
  selector:
    matchLabels:
      app: myapp
  minAvailable: 2
```

### GitOps với ArgoCD
```yaml
# Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: production
  source:
    repoURL: https://github.com/company/myapp-manifests
    targetRevision: HEAD
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

---
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
  - 'https://github.com/company/*'
  destinations:
  - namespace: 'production'
    server: https://kubernetes.default.svc
  - namespace: 'monitoring'
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  namespaceResourceWhitelist:
  - group: 'apps'
    kind: Deployment
  - group: ''
    kind: Service
  - group: ''
    kind: ConfigMap
  - group: ''
    kind: Secret
  roles:
  - name: admin
    description: Admin role
    policies:
    - p, proj:production:admin, applications, *, production/*, allow
    groups:
    - company:production-admins
```

### Istio Service Mesh
```yaml
# Gateway
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: myapp-gateway
  namespace: production
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - myapp.example.com
    tls:
      httpsRedirect: true
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: myapp-tls
    hosts:
    - myapp.example.com

---
# Virtual Service
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vs
  namespace: production
spec:
  hosts:
  - myapp.example.com
  gateways:
  - myapp-gateway
  http:
  - match:
    - uri:
        prefix: /api/v1
    route:
    - destination:
        host: myapp-service
        port:
          number: 80
      weight: 90
    - destination:
        host: myapp-service-canary
        port:
          number: 80
      weight: 10
    timeout: 30s
    retries:
      attempts: 3
      perTryTimeout: 10s

---
# Destination Rule
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-dr
  namespace: production
spec:
  host: myapp-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 10
    circuitBreaker:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

### Infrastructure as Code (Terraform)
```hcl
# main.tf
provider "aws" {
  region = "us-east-1"
}

resource "aws_ecs_cluster" "main" {
  name = "myapp-cluster"
}

resource "aws_ecs_task_definition" "app" {
  family                   = "myapp-task"
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 256
  memory                   = 512
  execution_role_arn       = aws_iam_role.ecs_task_execution_role.arn

  container_definitions = jsonencode([
    {
      name      = "myapp"
      image     = "myapp:latest"
      essential = true
      
      portMappings = [
        {
          containerPort = 3000
          hostPort      = 3000
        }
      ]
      
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = "/ecs/myapp"
          "awslogs-region"        = "us-east-1"
          "awslogs-stream-prefix" = "ecs"
        }
      }
    }
  ])
}

resource "aws_ecs_service" "app" {
  name            = "myapp-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 2

  network_configuration {
    subnets         = aws_subnet.private[*].id
    security_groups = [aws_security_group.ecs_tasks.id]
  }
}
```

## Monitoring và Logging

### Application Monitoring
```js
// Prometheus metrics
const promClient = require('prom-client');
const express = require('express');

// Create metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status']
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const labels = {
      method: req.method,
      route: req.route?.path || req.path,
      status: res.statusCode
    };
    
    httpRequestDuration.observe(labels, duration);
    httpRequestTotal.inc(labels);
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(promClient.register.metrics());
});
```

### Docker Compose với Monitoring
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - app-network

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - app-network

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
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

## Security Best Practices

### Docker Security - Advanced
```dockerfile
# Production-grade security Dockerfile
FROM node:18-alpine AS base

# Install security updates và remove package cache
RUN apk update && apk upgrade && \
    apk add --no-cache dumb-init && \
    rm -rf /var/cache/apk/*

# Create non-root user với specific UID/GID
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001 -G nodejs

# Set security-focused work directory
WORKDIR /app
RUN chown nextjs:nodejs /app

FROM base AS dependencies
# Copy package files
COPY package*.json ./
# Install dependencies với security audit
RUN npm ci --only=production && \
    npm audit --audit-level high && \
    npm cache clean --force

FROM base AS runtime
# Copy dependencies từ previous stage
COPY --from=dependencies --chown=nextjs:nodejs /app/node_modules ./node_modules

# Copy source code với proper ownership
COPY --chown=nextjs:nodejs . .

# Remove sensitive files
RUN rm -rf .git .gitignore README.md docs/ tests/

# Set strict file permissions
RUN chmod -R 755 /app && \
    chmod 644 package*.json

# Switch to non-root user
USER nextjs

# Use dumb-init as PID 1
ENTRYPOINT ["dumb-init", "--"]

# Health check với timeout
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# Expose port
EXPOSE 3000

CMD ["node", "server.js"]
```

### Container Security Scanning
```yaml
# Advanced security scanning trong CI/CD
name: Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    # Dependency vulnerability scanning
    - name: Run npm audit
      run: npm audit --audit-level high
    
    # SAST scanning với Semgrep
    - name: Semgrep SAST
      uses: returntocorp/semgrep-action@v1
      with:
        config: p/security-audit p/secrets p/owasp-top-ten
    
    # Build Docker image
    - name: Build Docker image
      run: docker build -t myapp:test .
    
    # Container image scanning với Trivy
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'myapp:test'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    # Upload results to GitHub Security
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
    
    # Container benchmarking với Docker Bench
    - name: Docker Bench Security
      run: |
        docker run --rm --net host --pid host --userns host --cap-add audit_control \
        -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
        -v /etc:/etc:ro \
        -v /usr/bin/containerd:/usr/bin/containerd:ro \
        -v /usr/bin/runc:/usr/bin/runc:ro \
        -v /usr/lib/systemd:/usr/lib/systemd:ro \
        -v /var/lib:/var/lib:ro \
        -v /var/run/docker.sock:/var/run/docker.sock:ro \
        --label docker_bench_security \
        docker/docker-bench-security
```

### Secrets Management - Enterprise
```yaml
# Multi-environment secrets management
name: Deploy with Vault

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    # Authenticate với HashiCorp Vault
    - name: Import Secrets from Vault
      uses: hashicorp/vault-action@v2
      with:
        url: ${{ secrets.VAULT_URL }}
        token: ${{ secrets.VAULT_TOKEN }}
        secrets: |
          secret/data/prod database_url | DATABASE_URL ;
          secret/data/prod api_key | API_KEY ;
          secret/data/prod jwt_secret | JWT_SECRET

    # Sử dụng AWS Secrets Manager
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
        aws-region: us-east-1
    
    - name: Retrieve secrets from AWS
      run: |
        DB_PASSWORD=$(aws secretsmanager get-secret-value \
          --secret-id prod/database/password \
          --query SecretString --output text)
        echo "::add-mask::$DB_PASSWORD"
        echo "DB_PASSWORD=$DB_PASSWORD" >> $GITHUB_ENV
    
    # Deploy với encrypted secrets
    - name: Deploy to production
      env:
        DATABASE_URL: ${{ env.DATABASE_URL }}
        API_KEY: ${{ env.API_KEY }}
        DB_PASSWORD: ${{ env.DB_PASSWORD }}
      run: |
        envsubst < k8s/deployment.yml | kubectl apply -f -
```

### Runtime Security với Falco
```yaml
# Falco rules for runtime security
- rule: Sensitive Mount
  desc: Detect sensitive mount points
  condition: >
    spawned_process and container and
    (proc.args contains "/proc" or 
     proc.args contains "/var/run/docker.sock" or
     proc.args contains "/etc/passwd")
  output: >
    Sensitive mount detected (user=%user.name command=%proc.cmdline 
    container=%container.name image=%container.image.repository)
  priority: WARNING

- rule: Unexpected Network Activity
  desc: Detect unexpected outbound connections
  condition: >
    outbound_connection and container and
    not fd.typechar=4 and
    not fd.name endswith ".local" and
    not proc.name in (node, nginx, curl, wget)
  output: >
    Unexpected network connection (user=%user.name command=%proc.cmdline 
    container=%container.name dest=%fd.name)
  priority: WARNING
```

### Kubernetes Security Policies
```yaml
# Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
# Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  - to: []
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 80
```

## Câu hỏi phỏng vấn thường gặp 2025

### Docker - Senior Level
1. **Container vs Virtual Machine khác nhau như thế nào? So sánh performance và resource usage**
   - Containers share OS kernel, VMs có riêng OS
   - Containers startup nhanh hơn (seconds vs minutes)
   - Memory footprint nhỏ hơn, CPU overhead thấp hơn

2. **Giải thích Docker layers và caching. Build optimization strategies?**
   - Layer caching: Docker cache từng instruction
   - Multi-stage builds để giảm final image size
   - .dockerignore để exclude unnecessary files
   - Order instructions từ ít thay đổi đến nhiều thay đổi

3. **Distroless images và Alpine images - ưu nhược điểm?**
   - Distroless: No shell, package manager → security tốt hơn
   - Alpine: Minimal Linux, musl libc thay vì glibc
   - Trade-off: security vs debugging capabilities

4. **Container orchestration: Kubernetes vs Docker Swarm vs ECS**
   - Kubernetes: Complex nhưng powerful, CNCF standard
   - Docker Swarm: Đơn giản, built-in Docker
   - ECS: AWS managed, integrate tốt với AWS services

5. **Docker networking deep dive - bridge, host, overlay networks**
   - Bridge: Default, isolated network
   - Host: Container dùng host network stack
   - Overlay: Multi-host networking cho Swarm

6. **Secrets management - Docker secrets vs external solutions**
   - Docker secrets: Built-in cho Swarm
   - External: HashiCorp Vault, AWS Secrets Manager
   - Kubernetes secrets với encryption at rest

### CI/CD - Advanced Topics
1. **GitOps workflow với ArgoCD/Flux**
   - Git as single source of truth
   - Declarative configuration management
   - Automated sync với desired state

2. **Progressive deployment strategies**
   - Canary: 5% → 25% → 50% → 100%
   - Blue-Green: Instant switchover
   - A/B testing với feature flags

3. **Pipeline optimization strategies**
   - Parallel execution, dependency management
   - Build caching (Docker layer cache, npm cache)
   - Artifact promotion across environments

4. **Security in CI/CD pipelines**
   - SAST/DAST scanning
   - Container image vulnerability scanning
   - Supply chain security (SBOM, signature verification)

5. **Branch protection và policy enforcement**
   - Required reviews, status checks
   - Automated security scanning
   - Compliance reporting

### Cloud Architecture - Production Ready
1. **Multi-region deployment strategies**
   - Active-Active vs Active-Passive
   - Data replication, consistency models
   - Latency optimization

2. **Auto-scaling patterns**
   - Horizontal Pod Autoscaler (HPA)
   - Vertical Pod Autoscaler (VPA)
   - Cluster Autoscaler
   - Predictive scaling vs reactive scaling

3. **Observability stack implementation**
   - Metrics: Prometheus + Grafana
   - Logs: ELK/EFK stack, Loki
   - Traces: Jaeger, Zipkin
   - APM: DataDog, New Relic

4. **Cost optimization techniques**
   - Right-sizing instances
   - Spot instances/preemptible VMs
   - Reserved instances planning
   - Resource tagging và chargeback

5. **Disaster Recovery & Business Continuity**
   - RTO/RPO requirements
   - Backup strategies (3-2-1 rule)
   - Cross-region replication
   - Chaos engineering practices

### Security - Zero Trust Model
1. **Container runtime security**
   - Runtime protection (Falco, Aqua)
   - Network policies implementation
   - Pod Security Standards
   - Service mesh security (Istio, Linkerd)

2. **Identity & Access Management**
   - RBAC implementation
   - Service-to-service authentication
   - JWT vs mTLS trade-offs
   - Identity federation

3. **Supply chain security**
   - Image signing và verification
   - Software Bill of Materials (SBOM)
   - Vulnerability scanning automation
   - Binary authorization policies

4. **Compliance frameworks**
   - SOC 2, ISO 27001, PCI-DSS
   - Data residency requirements
   - Audit logging implementation
   - Evidence collection automation

### Platform Engineering - 2025 Trends
1. **Internal Developer Platforms (IDP)**
   - Self-service infrastructure
   - Golden paths và templates
   - Developer experience metrics
   - Platform as Product mindset

2. **FinOps implementation**
   - Cost visibility và allocation
   - Budget alerts và governance
   - Resource optimization automation
   - Showback/chargeback models

3. **AI/ML workload orchestration**
   - GPU resource management
   - Model deployment pipelines
   - A/B testing for ML models
   - MLOps best practices

4. **Edge computing & CDN integration**
   - Edge deployment strategies
   - Content caching optimization
   - Latency-sensitive applications
   - Edge security considerations

## Cloud Services Comparison - 2025

### Container Services
| Service | AWS | Azure | GCP | 
|---------|-----|-------|-----|
| **Managed Kubernetes** | EKS | AKS | GKE |
| **Serverless Containers** | Fargate | Container Instances | Cloud Run |
| **Container Registry** | ECR | ACR | Artifact Registry |
| **Service Mesh** | App Mesh | Service Mesh | Anthos Service Mesh |

### CI/CD Services
| Feature | AWS | Azure | GCP |
|---------|-----|-------|-----|
| **Native CI/CD** | CodePipeline | DevOps Pipelines | Cloud Build |
| **Code Repository** | CodeCommit | Repos | Source Repositories |
| **Artifact Storage** | CodeArtifact | Artifacts | Artifact Registry |
| **Infrastructure as Code** | CloudFormation | ARM/Bicep | Deployment Manager |

### Monitoring & Observability
| Service | AWS | Azure | GCP |
|---------|-----|-------|-----|
| **Metrics** | CloudWatch | Monitor | Operations |
| **Logging** | CloudWatch Logs | Log Analytics | Cloud Logging |
| **Tracing** | X-Ray | Application Insights | Cloud Trace |
| **APM** | X-Ray | Application Insights | Cloud Profiler |

## Troubleshooting Common Issues

### Docker Troubleshooting
```bash
# Container won't start
docker logs <container-id>
docker inspect <container-id>

# Permission issues
docker run --user $(id -u):$(id -g) myapp

# Network connectivity
docker exec -it <container> ping <target>
docker network ls
docker network inspect bridge

# Disk space issues
docker system df
docker system prune -af
docker image prune -af
docker volume prune -f

# Build cache issues  
docker build --no-cache -t myapp .
docker builder prune

# Memory/CPU issues
docker stats
docker exec -it <container> top
docker exec -it <container> free -h
```

### Kubernetes Troubleshooting
```bash
# Pod issues
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl get events --sort-by='.lastTimestamp'

# Resource issues
kubectl top pods
kubectl top nodes
kubectl describe node <node-name>

# Network debugging
kubectl exec -it <pod> -- nslookup <service-name>
kubectl exec -it <pod> -- curl -v <service-name>:80

# Check resource quotas
kubectl describe resourcequota
kubectl get limitrange

# Service connectivity
kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints <service-name>

# Ingress debugging
kubectl describe ingress <ingress-name>
kubectl get ingress -o yaml

# ConfigMap/Secret issues
kubectl describe configmap <cm-name>
kubectl get secret <secret-name> -o yaml
```

### CI/CD Pipeline Debugging
```yaml
# GitHub Actions debugging
- name: Debug Environment
  run: |
    echo "Node version: $(node --version)"
    echo "NPM version: $(npm --version)"
    echo "Working directory: $(pwd)"
    echo "Files: $(ls -la)"
    env | sort

# Docker build debugging trong pipeline
- name: Debug Docker Build
  run: |
    docker version
    docker info
    docker build --progress=plain --no-cache -t myapp .

# Resource monitoring trong CI
- name: Monitor Resources
  run: |
    free -h
    df -h
    docker system df
```

## Performance Optimization Tips

### Container Optimization
- Sử dụng Alpine Linux base images
- Multi-stage builds để giảm size
- .dockerignore để exclude unnecessary files
- Layer ordering optimization
- Use specific versions, không dùng `latest`

### Kubernetes Optimization
- Resource requests/limits chính xác
- Pod disruption budgets
- Node affinity rules
- Horizontal Pod Autoscaler tuning
- Vertical Pod Autoscaler cho right-sizing

### CI/CD Optimization
- Parallel job execution
- Build cache strategies
- Artifact caching
- Docker layer caching
- Matrix builds cho multiple environments

## Practical Interview Tasks - Advanced
1. **Design và implement GitOps workflow với ArgoCD**
2. **Setup monitoring stack (Prometheus/Grafana/AlertManager)**
3. **Implement canary deployment với Istio**
4. **Create Terraform modules cho multi-environment**
5. **Setup RBAC và security policies trong Kubernetes**
6. **Design disaster recovery strategy cho containerized apps**
7. **Implement cost optimization strategies**
8. **Setup secrets management với HashiCorp Vault**
9. **Create custom Kubernetes operators**
10. **Implement chaos engineering với Chaos Monkey**
