# Câu Hỏi Phỏng Vấn Fullstack / DevOps

## 1. Mô tả kiến trúc fullstack bạn đã từng xây dựng (frontend – backend – database – deployment)

### Kiến trúc tổng quan:
```
Frontend (React/Next.js) → Load Balancer → Backend API (Node.js/Express) → Database (PostgreSQL/MongoDB) → Cloud Services
```

### Chi tiết từng layer:

#### Frontend:
- **Framework**: React/Next.js với TypeScript
- **State Management**: Redux Toolkit/Zustand
- **UI Libraries**: Material-UI/Tailwind CSS
- **Build Tools**: Vite/Webpack
- **Deployment**: Vercel/Netlify/CloudFlare Pages

#### Backend:
- **Runtime**: Node.js với Express/Fastify hoặc NestJS
- **Language**: TypeScript
- **Authentication**: JWT với refresh token
- **API Style**: RESTful API + GraphQL (tùy dự án)
- **Middleware**: CORS, Rate limiting, Compression, Security headers
- **Validation**: Joi/Zod cho request validation

#### Database:
- **Primary DB**: PostgreSQL với Prisma ORM
- **Caching**: Redis cho session và caching
- **Search**: Elasticsearch cho full-text search
- **File Storage**: AWS S3/CloudFlare R2

#### Deployment & Infrastructure:
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes (production) hoặc Docker Swarm
- **Cloud Provider**: AWS/Google Cloud/Azure
- **CDN**: CloudFlare
- **Load Balancer**: Nginx/HAProxy
- **Monitoring**: Prometheus + Grafana

### Ví dụ kiến trúc cụ thể:
```
[User] → [CloudFlare CDN] → [Nginx Load Balancer] 
        ↓
[React App (Vercel)] → [Node.js API (AWS ECS)] → [PostgreSQL (AWS RDS)]
                                                ↓
                                        [Redis Cache (AWS ElastiCache)]
                                                ↓
                                        [S3 File Storage]
```

## 2. CI/CD pipeline cơ bản gồm những bước gì?

### Pipeline stages:

#### 1. Source Code Management
- **Git hooks**: Pre-commit hooks chạy linting
- **Branching strategy**: GitFlow hoặc GitHub Flow
- **Code review**: Pull Request mandatory

#### 2. Build Stage
```yaml
# .github/workflows/ci.yml
- name: Install dependencies
  run: npm install
  
- name: Lint code
  run: npm run lint
  
- name: Type check
  run: npm run type-check
  
- name: Run tests
  run: npm run test:coverage
  
- name: Build application
  run: npm run build
```

#### 3. Test Stage
- **Unit Tests**: Jest/Vitest
- **Integration Tests**: Supertest cho API
- **E2E Tests**: Playwright/Cypress
- **Security Tests**: npm audit, Snyk
- **Code Coverage**: Minimum 80%

#### 4. Security & Quality Gates
- **SAST**: SonarQube static analysis
- **Dependency Scanning**: Dependabot
- **Container Scanning**: Trivy/Clair
- **Quality Gates**: Code coverage, complexity metrics

#### 5. Build & Package
- **Docker Build**: Multi-stage builds
- **Image Registry**: AWS ECR/Docker Hub
- **Versioning**: Semantic versioning

#### 6. Deployment Stages
```yaml
# Deployment pipeline
Development → Staging → Production

# Blue-Green deployment strategy
- Deploy to green environment
- Run smoke tests
- Switch traffic from blue to green
- Keep blue as rollback option
```

#### 7. Post-Deployment
- **Health Checks**: Liveness/readiness probes
- **Monitoring**: Application metrics
- **Notifications**: Slack/Email alerts

### Pipeline tools:
- **GitHub Actions/GitLab CI**: Pipeline orchestration
- **Docker**: Containerization
- **Terraform**: Infrastructure as Code
- **ArgoCD**: GitOps deployment

## 3. Bạn xử lý logs trong production như thế nào?

### Log Strategy Framework:

#### 1. Log Levels & Structure
```javascript
// Structured logging với Winston
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { 
    service: 'user-service',
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION
  }
});

// Log với correlation ID
logger.info('User created', {
  correlationId: req.headers['x-correlation-id'],
  userId: user.id,
  email: user.email,
  timestamp: new Date().toISOString()
});
```

#### 2. Log Categories
- **Application logs**: Business logic events
- **Access logs**: HTTP requests/responses
- **Error logs**: Exceptions và errors
- **Audit logs**: User actions, data changes
- **Performance logs**: Response times, resource usage

#### 3. Log Aggregation & Storage
```yaml
# ELK Stack setup
Elasticsearch: Storage và search
Logstash: Log processing và parsing  
Kibana: Visualization và dashboards

# Modern alternative: EFK Stack
Fluentd: Log collection
Elasticsearch: Storage
Kibana: Visualization
```

#### 4. Log Processing Pipeline
```
Application → Filebeat → Logstash → Elasticsearch → Kibana
             ↓
         [Log Rotation]
```

#### 5. Log Management Best Practices
- **Retention policy**: 30 days cho debug logs, 1 năm cho audit logs
- **Log rotation**: Logrotate để tránh disk full
- **Sensitive data**: Mask PII, passwords, API keys
- **Correlation IDs**: Track requests across services
- **Structured format**: JSON format cho easy parsing

#### 6. Alerting Rules
```javascript
// Ví dụ alert rules
- Error rate > 5% trong 5 phút
- Response time > 2s trong 10 phút  
- Disk usage > 85%
- Memory usage > 90%
- 5xx errors > 10/minute
```

## 4. Monitoring & alerting: dùng công cụ gì?

### Monitoring Stack:

#### 1. Metrics Collection
```yaml
# Prometheus setup
- Node Exporter: System metrics (CPU, memory, disk)
- Application metrics: Custom business metrics
- Blackbox Exporter: HTTP/HTTPS uptime monitoring
- cAdvisor: Container metrics
```

#### 2. Application Performance Monitoring (APM)
- **New Relic/DataDog**: Full-stack monitoring
- **Sentry**: Error tracking và performance
- **Prometheus**: Metrics collection
- **Jaeger/Zipkin**: Distributed tracing

#### 3. Visualization
```yaml
# Grafana Dashboards
- Infrastructure metrics: CPU, Memory, Disk, Network
- Application metrics: Request rate, Response time, Error rate  
- Business metrics: User registrations, Orders, Revenue
- Database metrics: Connection pool, Query performance
```

#### 4. Alerting Strategy
```yaml
# Alertmanager rules
- Severity levels: Critical, Warning, Info
- Escalation paths: 
  - Critical: Immediately → On-call engineer
  - Warning: 15 minutes → Team lead
  - Info: Dashboard only

# Alert channels:
- PagerDuty: Critical production issues
- Slack: Team notifications
- Email: Weekly reports
```

#### 5. Health Checks & SLOs
```javascript
// Health check endpoints
app.get('/health', (req, res) => {
  const health = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now(),
    checks: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      external_api: await checkExternalAPI()
    }
  };
  res.status(200).json(health);
});

// SLOs (Service Level Objectives)
- Availability: 99.9% uptime
- Response time: P95 < 200ms
- Error rate: < 0.1%
```

#### 6. Monitoring Tools Stack
- **Infrastructure**: Prometheus + Grafana
- **Applications**: New Relic/DataDog
- **Logs**: ELK Stack
- **Uptime**: Pingdom/UptimeRobot
- **Synthetic monitoring**: Datadog Synthetics

## 5. Khi có traffic tăng gấp 10 lần, bạn sẽ scale hệ thống thế nào?

### Scaling Strategy:

#### 1. Immediate Actions (Minutes)
```yaml
# Horizontal scaling
- Auto Scaling Groups: Tăng instance count
- Container orchestration: Scale replicas
- Database: Read replicas
- CDN: Cache static assets
```

#### 2. Application Layer Scaling
```javascript
// Load balancing strategies
- Round Robin: Phân phối đều requests
- Least Connections: Route đến server ít connection nhất
- Sticky Sessions: User luôn được route đến cùng 1 server

// Horizontal scaling với PM2
pm2 start app.js -i max // Tận dụng tất cả CPU cores
```

#### 3. Database Scaling
```sql
-- Chiến lược scaling database
1. Read Replicas: Tách read/write operations
2. Database Sharding: Phân chia data theo region/user_id
3. Connection Pooling: Giới hạn concurrent connections
4. Query Optimization: Index, query caching
5. Database Caching: Redis cho frequently accessed data
```

#### 4. Caching Strategy
```javascript
// Multi-level caching
1. Browser Cache: Static assets (CSS, JS, images)
2. CDN Cache: Global content distribution  
3. Application Cache: Redis/Memcached
4. Database Cache: Query result caching

// Redis caching example
const cache = require('redis').createClient();
const getUserData = async (userId) => {
  const cacheKey = `user:${userId}`;
  let userData = await cache.get(cacheKey);
  
  if (!userData) {
    userData = await database.getUser(userId);
    await cache.setex(cacheKey, 3600, JSON.stringify(userData));
  }
  return JSON.parse(userData);
};
```

#### 5. Infrastructure Scaling
```yaml
# Auto Scaling Configuration
CloudWatch Alarms:
  - CPU > 70%: Scale out +2 instances
  - CPU < 30%: Scale in -1 instance
  - Memory > 85%: Scale out +1 instance

# Kubernetes HPA (Horizontal Pod Autoscaler)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 3
  maxReplicas: 100
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

#### 6. Microservices & Architecture
```javascript
// Service decomposition
Monolith → Microservices
- User Service: Authentication, user management
- Order Service: Order processing
- Payment Service: Payment handling  
- Notification Service: Email/SMS notifications

// API Gateway pattern
const gateway = {
  '/api/users/*': 'user-service:3001',
  '/api/orders/*': 'order-service:3002', 
  '/api/payments/*': 'payment-service:3003'
};
```

#### 7. Performance Optimization
```javascript
// Code-level optimizations
1. Database queries: N+1 problem resolution
2. Async processing: Queue heavy operations
3. Connection pooling: Reuse database connections
4. Compression: Gzip response compression
5. Lazy loading: Load data on demand

// Example: Async queue processing
const Queue = require('bull');
const emailQueue = new Queue('email sending');

// Producer: Add job to queue instead of processing immediately  
emailQueue.add('send welcome email', {
  userId: user.id,
  email: user.email
});

// Consumer: Process jobs asynchronously
emailQueue.process('send welcome email', async (job) => {
  await sendWelcomeEmail(job.data);
});
```

#### 8. Monitoring During Scale
```yaml
# Key metrics to watch:
- Response time: Should remain < 200ms P95
- Error rate: Should stay < 0.1%  
- CPU/Memory: Should not exceed 80%
- Database connections: Monitor connection pool
- Queue length: Background job processing
- Cost: Monitor AWS/GCP spending
```

### Scaling Timeline:
- **0-5 minutes**: Enable auto-scaling, increase instances
- **5-30 minutes**: Database read replicas, CDN optimization
- **30 minutes - 2 hours**: Code optimizations, caching implementation
- **2+ hours**: Architecture changes, microservices decomposition

---

*Lưu ý: Các câu trả lời trên dựa trên kinh nghiệm thực tế và best practices trong ngành. Trong phỏng vấn, nên điều chỉnh câu trả lời theo context cụ thể của công ty và vị trí ứng tuyển.*
