# 12. Deployment & Monitoring
- **PM2**: Quản lý process Node.js, tự restart khi crash, scale nhiều instance, log, monitoring.
  - Cài đặt: `npm install -g pm2`
  - Chạy app: `pm2 start index.js`
  - Xem log: `pm2 logs`
  - Scale: `pm2 scale index 4` (chạy 4 instance)
  - Tự động restart khi server reboot: `pm2 startup`

- **Log**: Ghi log ra file hoặc console để theo dõi hoạt động, lỗi (dùng winston, morgan, ...).
  - Ví dụ dùng winston:
    ```js
    const winston = require('winston');
    const logger = winston.createLogger({
      transports: [new winston.transports.File({ filename: 'app.log' })]
    });
    logger.info('App started');
    ```

- **Healthcheck**: Tạo endpoint `/health` trả về trạng thái app (OK, DB connect, ...), giúp load balancer/monitor kiểm tra app còn sống.
  - Ví dụ:
    ```js
    app.get('/health', (req, res) => res.send('OK'));
    ```

## Docker & Containerization
- **Dockerfile cho Node.js:**
  ```dockerfile
  FROM node:18-alpine
  
  WORKDIR /app
  
  # Copy package files first (for better caching)
  COPY package*.json ./
  RUN npm ci --only=production
  
  # Copy source code
  COPY . .
  
  # Create non-root user
  RUN addgroup -g 1001 -S nodejs
  RUN adduser -S nextjs -u 1001
  USER nextjs
  
  EXPOSE 3000
  
  CMD ["node", "index.js"]
  ```

- **Docker Compose:**
  ```yaml
  # docker-compose.yml
  version: '3.8'
  services:
    app:
      build: .
      ports:
        - "3000:3000"
      environment:
        - NODE_ENV=production
      depends_on:
        - db
        - redis
    
    db:
      image: postgres:14
      environment:
        POSTGRES_DB: myapp
        POSTGRES_PASSWORD: password
      volumes:
        - postgres_data:/var/lib/postgresql/data
    
    redis:
      image: redis:7-alpine
      
  volumes:
    postgres_data:
  ```

## Cloud Platform Deployment
- **AWS Deployment:**
  ```js
  // Elastic Beanstalk
  // 1. Install EB CLI: pip install awsebcli
  // 2. Initialize: eb init
  // 3. Deploy: eb create production
  
  // EC2 with Load Balancer
  // 1. Create EC2 instances
  // 2. Setup Application Load Balancer
  // 3. Configure Auto Scaling Group
  // 4. Setup CloudWatch monitoring
  ```

- **Vercel (Next.js):**
  ```json
  // vercel.json
  {
    "version": 2,
    "builds": [
      {
        "src": "package.json",
        "use": "@vercel/next"
      }
    ],
    "env": {
      "DATABASE_URL": "@database-url"
    }
  }
  ```

- **Railway/Render:**
  ```yaml
  # railway.json
  {
    "$schema": "https://railway.app/railway.schema.json",
    "build": {
      "builder": "NIXPACKS"
    },
    "deploy": {
      "startCommand": "npm start",
      "healthcheckPath": "/health"
    }
  }
  ```

## CI/CD Pipeline
- **GitHub Actions:**
  ```yaml
  # .github/workflows/deploy.yml
  name: Deploy to Production
  
  on:
    push:
      branches: [main]
  
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - uses: actions/setup-node@v3
          with:
            node-version: '18'
            cache: 'npm'
        - run: npm ci
        - run: npm test
        - run: npm run build
    
    deploy:
      needs: test
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - name: Deploy to production
          run: |
            docker build -t myapp .
            docker tag myapp registry.heroku.com/myapp/web
            docker push registry.heroku.com/myapp/web
            heroku container:release web -a myapp
  ```

- **GitLab CI:**
  ```yaml
  # .gitlab-ci.yml
  stages:
    - test
    - build
    - deploy
  
  test:
    stage: test
    image: node:18
    script:
      - npm ci
      - npm test
      - npm run lint
  
  build:
    stage: build
    image: docker:latest
    script:
      - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
      - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  
  deploy:
    stage: deploy
    script:
      - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  ```

## Environment Management
- **Environment Configuration:**
  ```js
  // config/config.js
  const config = {
    development: {
      port: 3000,
      db: {
        host: 'localhost',
        database: 'myapp_dev'
      },
      logging: true
    },
    production: {
      port: process.env.PORT || 8080,
      db: {
        host: process.env.DB_HOST,
        database: process.env.DB_NAME
      },
      logging: false
    }
  };
  
  module.exports = config[process.env.NODE_ENV || 'development'];
  ```

- **Secrets Management:**
  ```js
  // Sử dụng AWS Secrets Manager
  const AWS = require('aws-sdk');
  const secretsManager = new AWS.SecretsManager();
  
  const getSecret = async (secretName) => {
    const result = await secretsManager.getSecretValue({ 
      SecretId: secretName 
    }).promise();
    return JSON.parse(result.SecretString);
  };
  
  // Hoặc dùng HashiCorp Vault
  const vault = require('node-vault')();
  const secrets = await vault.read('secret/data/myapp');
  ```

## Load Balancing & Scaling
- **Nginx Load Balancer:**
  ```nginx
  upstream app {
      server app1:3000;
      server app2:3000;
      server app3:3000;
  }
  
  server {
      listen 80;
      location / {
          proxy_pass http://app;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
      }
  }
  ```

- **Horizontal Scaling:**
  ```yaml
  # Kubernetes deployment
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: myapp
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: myapp
    template:
      metadata:
        labels:
          app: myapp
      spec:
        containers:
        - name: myapp
          image: myapp:latest
          ports:
          - containerPort: 3000
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
  ```

## Comprehensive Logging
- **Structured Logging:**
  ```js
  const winston = require('winston');
  
  const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
      winston.format.timestamp(),
      winston.format.errors({ stack: true }),
      winston.format.json()
    ),
    defaultMeta: { service: 'user-service' },
    transports: [
      new winston.transports.File({ filename: 'error.log', level: 'error' }),
      new winston.transports.File({ filename: 'combined.log' }),
      new winston.transports.Console({
        format: winston.format.simple()
      })
    ]
  });
  
  // Usage
  logger.info('User login successful', { userId: 123, ip: '192.168.1.1' });
  logger.error('Database connection failed', { error: err.message, stack: err.stack });
  ```

- **Request Logging Middleware:**
  ```js
  const morgan = require('morgan');
  
  // Custom format
  morgan.token('user-id', (req) => req.user?.id || 'anonymous');
  
  app.use(morgan(':method :url :status :res[content-length] - :response-time ms :user-id'));
  
  // JSON format for production
  app.use(morgan((tokens, req, res) => {
    return JSON.stringify({
      method: tokens.method(req, res),
      url: tokens.url(req, res),
      status: tokens.status(req, res),
      contentLength: tokens.res(req, res, 'content-length'),
      responseTime: tokens['response-time'](req, res),
      userAgent: tokens['user-agent'](req, res),
      timestamp: new Date().toISOString()
    });
  }));
  ```

## Advanced Monitoring & Alerting
- **Custom Metrics với Prometheus:**
  ```js
  const promClient = require('prom-client');
  
  // Create metrics
  const httpRequestDuration = new promClient.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status'],
    buckets: [0.1, 0.5, 1, 2, 5]
  });
  
  const activeUsers = new promClient.Gauge({
    name: 'active_users_total',
    help: 'Number of active users'
  });
  
  // Middleware to collect metrics
  app.use((req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
      const duration = (Date.now() - start) / 1000;
      httpRequestDuration
        .labels(req.method, req.route?.path || req.path, res.statusCode)
        .observe(duration);
    });
    
    next();
  });
  
  // Expose metrics
  app.get('/metrics', async (req, res) => {
    res.set('Content-Type', promClient.register.contentType);
    res.end(await promClient.register.metrics());
  });
  ```

- **Application Performance Monitoring:**
  ```js
  // New Relic setup
  require('newrelic');
  
  // Sentry error tracking
  const Sentry = require('@sentry/node');
  Sentry.init({ dsn: process.env.SENTRY_DSN });
  
  // Custom performance tracking
  const performanceMonitor = {
    startTimer: (operation) => {
      const start = process.hrtime.bigint();
      return () => {
        const duration = Number(process.hrtime.bigint() - start) / 1000000;
        logger.info(`${operation} completed`, { duration: `${duration}ms` });
        return duration;
      };
    }
  };
  
  // Usage
  const timer = performanceMonitor.startTimer('database-query');
  const result = await db.query('SELECT * FROM users');
  timer();
  ```

## Security in Deployment
- **HTTPS Configuration:**
  ```js
  const https = require('https');
  const fs = require('fs');
  
  const options = {
    key: fs.readFileSync('private-key.pem'),
    cert: fs.readFileSync('certificate.pem')
  };
  
  https.createServer(options, app).listen(443);
  
  // Redirect HTTP to HTTPS
  app.use((req, res, next) => {
    if (req.header('x-forwarded-proto') !== 'https') {
      return res.redirect(`https://${req.header('host')}${req.url}`);
    }
    next();
  });
  ```

- **Security Headers:**
  ```js
  const helmet = require('helmet');
  
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"]
      }
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true
    }
  }));
  ```

## Deployment Best Practices
1. **Blue-Green Deployment**: Chạy 2 environment, switch traffic khi deploy
2. **Rolling Updates**: Update từng server một cách tuần tự
3. **Canary Releases**: Deploy cho một phần nhỏ users trước
4. **Health Checks**: Kiểm tra app health trước khi route traffic
5. **Database Migrations**: Chạy migration trước khi deploy code
6. **Rollback Strategy**: Chuẩn bị kế hoạch rollback khi có lỗi
7. **Environment Parity**: Dev, staging, production giống nhau
8. **Immutable Infrastructure**: Không modify server đang chạy

## Monitoring Tools
- **APM**: New Relic, DataDog, AppDynamics
- **Error Tracking**: Sentry, Rollbar, Bugsnag  
- **Metrics**: Prometheus + Grafana, CloudWatch
- **Logs**: ELK Stack, Splunk, Fluentd
- **Uptime**: Pingdom, UptimeRobot, StatusCake
- **Performance**: GTmetrix, WebPageTest, Lighthouse CI
