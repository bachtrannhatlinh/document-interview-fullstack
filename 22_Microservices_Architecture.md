# MICROSERVICES ARCHITECTURE - INTERVIEW PREPARATION

## 1. What are Microservices?

### Definition
- **Architecture pattern** chia application thành các **loosely coupled services**
- Mỗi service chạy trong **own process** và communicate qua **well-defined APIs**
- Services are **independently deployable** và **technology agnostic**
- **Business capability focused** - mỗi service phục vụ một business domain

### Key Characteristics
- **Componentization via Services** - thay vì libraries
- **Organized around Business Capabilities** - cross-functional teams
- **Products not Projects** - "you build it, you run it"
- **Smart endpoints and dumb pipes** - decentralized
- **Decentralized Governance** - technology diversity
- **Failure Tolerant** - designed for failure

## 2. Monolith vs Microservices

### Monolithic Architecture
```
┌─────────────────────────────────┐
│         Monolithic App          │
├─────────────────────────────────┤
│  User Management | Orders       │
│  Products       | Payments      │
│  Inventory      | Notifications │
└─────────────────────────────────┘
         │
    ┌────▼────┐
    │Database │
    └─────────┘
```

**Pros:**
- Simple to develop, test, deploy initially
- Good performance (no network calls)
- Easy transactions (ACID)
- Simple debugging

**Cons:**
- Scaling issues (scale entire app)
- Technology lock-in
- Large team coordination problems
- Risk of system failure

### Microservices Architecture
```
┌─────────┐   ┌─────────┐   ┌──────────┐
│  User   │   │ Product │   │  Order   │
│Service  │   │Service  │   │ Service  │
└────┬────┘   └────┬────┘   └────┬─────┘
     │             │             │
┌────▼────┐   ┌────▼────┐   ┌────▼─────┐
│User DB  │   │Prod DB  │   │Order DB  │
└─────────┘   └─────────┘   └──────────┘
```

**Pros:**
- Independent scaling
- Technology diversity
- Fault isolation
- Team autonomy
- Easier to understand (smaller codebase)

**Cons:**
- Distributed system complexity
- Network communication overhead
- Data consistency challenges
- Testing complexity
- Operational overhead

## 3. Service Decomposition Strategies

### Domain-Driven Design (DDD)
```javascript
// E-commerce example - Bounded Contexts
const boundedContexts = {
  userManagement: {
    entities: ['User', 'Profile', 'Authentication'],
    services: ['UserService', 'AuthService'],
    responsibilities: ['user registration', 'login', 'profile management']
  },
  
  catalog: {
    entities: ['Product', 'Category', 'Inventory'],
    services: ['ProductService', 'CatalogService'],
    responsibilities: ['product management', 'search', 'inventory']
  },
  
  ordering: {
    entities: ['Order', 'OrderItem', 'Cart'],
    services: ['OrderService', 'CartService'],
    responsibilities: ['order processing', 'cart management']
  },
  
  payment: {
    entities: ['Payment', 'Transaction', 'Invoice'],
    services: ['PaymentService', 'BillingService'],
    responsibilities: ['payment processing', 'billing']
  }
};
```

### Decomposition Patterns

#### 1. Decompose by Business Capability
```javascript
// User Service
class UserService {
  async createUser(userData) {
    // Handle user creation
    const user = await this.userRepository.create(userData);
    
    // Publish domain event
    await this.eventBus.publish('user.created', {
      userId: user.id,
      email: user.email
    });
    
    return user;
  }
}

// Order Service  
class OrderService {
  async createOrder(orderData) {
    // Validate user exists (via API call or event)
    const user = await this.userServiceClient.getUser(orderData.userId);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const order = await this.orderRepository.create(orderData);
    
    // Publish order created event
    await this.eventBus.publish('order.created', {
      orderId: order.id,
      userId: order.userId,
      total: order.total
    });
    
    return order;
  }
}
```

#### 2. Decompose by Data Model
```javascript
// Each service owns its data
// User Service Database Schema
const userSchema = {
  users: {
    id: 'UUID',
    email: 'STRING',
    name: 'STRING',
    created_at: 'TIMESTAMP'
  },
  user_profiles: {
    user_id: 'UUID',
    avatar: 'STRING',
    preferences: 'JSON'
  }
};

// Product Service Database Schema
const productSchema = {
  products: {
    id: 'UUID',
    name: 'STRING',
    price: 'DECIMAL',
    category_id: 'UUID'
  },
  categories: {
    id: 'UUID',
    name: 'STRING',
    description: 'TEXT'
  }
};
```

## 4. Communication Patterns

### Synchronous Communication

#### REST APIs
```javascript
// User Service API
class UserController {
  async getUser(req, res) {
    const { userId } = req.params;
    
    try {
      const user = await this.userService.getUserById(userId);
      res.json(user);
    } catch (error) {
      res.status(404).json({ error: 'User not found' });
    }
  }
}

// Order Service calling User Service
class OrderService {
  constructor(userServiceClient) {
    this.userServiceClient = userServiceClient;
  }
  
  async createOrder(orderData) {
    // Synchronous call to User Service
    try {
      const user = await this.userServiceClient.get(`/users/${orderData.userId}`);
      
      if (!user) {
        throw new Error('Invalid user');
      }
      
      return await this.processOrder(orderData);
    } catch (error) {
      throw new Error(`Order creation failed: ${error.message}`);
    }
  }
}
```

#### gRPC Communication
```protobuf
// user.proto
syntax = "proto3";

service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
}

message GetUserRequest {
  string user_id = 1;
}

message GetUserResponse {
  string id = 1;
  string email = 2;
  string name = 3;
}
```

```javascript
// gRPC Client implementation
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

class UserServiceClient {
  constructor() {
    const packageDefinition = protoLoader.loadSync('user.proto');
    const userProto = grpc.loadPackageDefinition(packageDefinition);
    
    this.client = new userProto.UserService(
      'user-service:50051',
      grpc.credentials.createInsecure()
    );
  }
  
  async getUser(userId) {
    return new Promise((resolve, reject) => {
      this.client.GetUser({ user_id: userId }, (error, response) => {
        if (error) {
          reject(error);
        } else {
          resolve(response);
        }
      });
    });
  }
}
```

### Asynchronous Communication

#### Message Queues (RabbitMQ)
```javascript
// Event Publisher
class EventPublisher {
  constructor(channel) {
    this.channel = channel;
  }
  
  async publishUserCreated(userData) {
    const event = {
      eventType: 'user.created',
      timestamp: new Date().toISOString(),
      data: userData
    };
    
    await this.channel.publish('user.events', 'user.created', 
      Buffer.from(JSON.stringify(event))
    );
  }
}

// Event Consumer
class OrderEventHandler {
  constructor(orderService) {
    this.orderService = orderService;
  }
  
  async handleUserCreated(eventData) {
    console.log('User created event received:', eventData);
    
    // Maybe create a shopping cart for new user
    await this.orderService.createUserCart(eventData.userId);
  }
  
  async setupConsumers(channel) {
    await channel.consume('order.user.events', async (msg) => {
      const event = JSON.parse(msg.content.toString());
      
      switch (event.eventType) {
        case 'user.created':
          await this.handleUserCreated(event.data);
          break;
      }
      
      channel.ack(msg);
    });
  }
}
```

#### Apache Kafka
```javascript
// Kafka Producer
const kafka = require('kafkajs');

class EventProducer {
  constructor() {
    this.kafka = kafka({
      clientId: 'user-service',
      brokers: ['localhost:9092']
    });
    this.producer = this.kafka.producer();
  }
  
  async publishEvent(topic, event) {
    await this.producer.send({
      topic,
      messages: [{
        key: event.aggregateId,
        value: JSON.stringify(event),
        timestamp: Date.now()
      }]
    });
  }
  
  async publishUserCreated(userId, userData) {
    await this.publishEvent('user-events', {
      eventType: 'UserCreated',
      aggregateId: userId,
      data: userData,
      version: 1
    });
  }
}

// Kafka Consumer
class EventConsumer {
  constructor() {
    this.kafka = kafka({
      clientId: 'order-service',
      brokers: ['localhost:9092']
    });
    this.consumer = this.kafka.consumer({ groupId: 'order-service-group' });
  }
  
  async startConsuming() {
    await this.consumer.subscribe({ topic: 'user-events' });
    
    await this.consumer.run({
      eachMessage: async ({ topic, partition, message }) => {
        const event = JSON.parse(message.value.toString());
        
        switch (event.eventType) {
          case 'UserCreated':
            await this.handleUserCreated(event);
            break;
        }
      }
    });
  }
}
```

## 5. Core Patterns & Solutions

### API Gateway Pattern
```javascript
// API Gateway Implementation
const express = require('express');
const httpProxy = require('http-proxy-middleware');

class ApiGateway {
  constructor() {
    this.app = express();
    this.setupRoutes();
    this.setupMiddleware();
  }
  
  setupMiddleware() {
    // Rate limiting
    this.app.use(rateLimiter({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 100 // limit each IP to 100 requests per windowMs
    }));
    
    // Authentication
    this.app.use(this.authenticateRequest);
    
    // Request logging
    this.app.use(this.logRequest);
  }
  
  setupRoutes() {
    // User Service routes
    this.app.use('/api/users', httpProxy({
      target: 'http://user-service:3001',
      changeOrigin: true,
      onError: this.handleServiceError
    }));
    
    // Product Service routes
    this.app.use('/api/products', httpProxy({
      target: 'http://product-service:3002',
      changeOrigin: true,
      onError: this.handleServiceError
    }));
    
    // Order Service routes
    this.app.use('/api/orders', httpProxy({
      target: 'http://order-service:3003',
      changeOrigin: true,
      onError: this.handleServiceError
    }));
  }
  
  authenticateRequest = async (req, res, next) => {
    const token = req.headers.authorization;
    
    if (!token) {
      return res.status(401).json({ error: 'No token provided' });
    }
    
    try {
      const user = await this.authService.verifyToken(token);
      req.user = user;
      next();
    } catch (error) {
      res.status(401).json({ error: 'Invalid token' });
    }
  };
  
  handleServiceError = (err, req, res) => {
    console.error(`Service error for ${req.url}:`, err);
    res.status(503).json({ 
      error: 'Service temporarily unavailable' 
    });
  };
}
```

### Circuit Breaker Pattern
```javascript
class CircuitBreaker {
  constructor(service, options = {}) {
    this.service = service;
    this.failureThreshold = options.failureThreshold || 5;
    this.recoveryTimeout = options.recoveryTimeout || 60000;
    this.monitoringPeriod = options.monitoringPeriod || 10000;
    
    this.state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.successCount = 0;
  }
  
  async execute(operation, ...args) {
    if (this.state === 'OPEN') {
      if (this.shouldAttemptReset()) {
        this.state = 'HALF_OPEN';
        this.successCount = 0;
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await operation.apply(this.service, args);
      
      if (this.state === 'HALF_OPEN') {
        this.successCount++;
        if (this.successCount >= 3) {
          this.reset();
        }
      }
      
      return result;
    } catch (error) {
      this.recordFailure();
      throw error;
    }
  }
  
  recordFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
  
  shouldAttemptReset() {
    return Date.now() - this.lastFailureTime >= this.recoveryTimeout;
  }
  
  reset() {
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.lastFailureTime = null;
  }
}

// Usage
class OrderService {
  constructor() {
    this.userServiceClient = new UserServiceClient();
    this.userServiceCircuitBreaker = new CircuitBreaker(
      this.userServiceClient, 
      { failureThreshold: 3, recoveryTimeout: 30000 }
    );
  }
  
  async createOrder(orderData) {
    try {
      // Use circuit breaker for external service calls
      const user = await this.userServiceCircuitBreaker.execute(
        this.userServiceClient.getUser,
        orderData.userId
      );
      
      return await this.processOrder(orderData, user);
    } catch (error) {
      // Handle circuit breaker or service errors
      if (error.message === 'Circuit breaker is OPEN') {
        // Use cached data or default behavior
        return await this.processOrderWithoutUserValidation(orderData);
      }
      throw error;
    }
  }
}
```

### Service Discovery
```javascript
// Service Registry
class ServiceRegistry {
  constructor() {
    this.services = new Map();
  }
  
  register(serviceName, serviceInfo) {
    if (!this.services.has(serviceName)) {
      this.services.set(serviceName, []);
    }
    
    this.services.get(serviceName).push({
      ...serviceInfo,
      registeredAt: Date.now(),
      lastHeartbeat: Date.now()
    });
    
    console.log(`Service ${serviceName} registered:`, serviceInfo);
  }
  
  discover(serviceName) {
    const instances = this.services.get(serviceName) || [];
    
    // Filter healthy instances (received heartbeat in last 30s)
    const healthyInstances = instances.filter(
      instance => Date.now() - instance.lastHeartbeat < 30000
    );
    
    // Load balancing - round robin
    if (healthyInstances.length > 0) {
      const index = Math.floor(Math.random() * healthyInstances.length);
      return healthyInstances[index];
    }
    
    throw new Error(`No healthy instances found for service: ${serviceName}`);
  }
  
  heartbeat(serviceName, serviceId) {
    const instances = this.services.get(serviceName) || [];
    const instance = instances.find(inst => inst.id === serviceId);
    
    if (instance) {
      instance.lastHeartbeat = Date.now();
    }
  }
}

// Service Discovery Client
class ServiceClient {
  constructor(serviceRegistry) {
    this.serviceRegistry = serviceRegistry;
    this.httpClient = axios.create();
  }
  
  async callService(serviceName, endpoint, options = {}) {
    try {
      const serviceInstance = this.serviceRegistry.discover(serviceName);
      const url = `http://${serviceInstance.host}:${serviceInstance.port}${endpoint}`;
      
      return await this.httpClient.request({
        url,
        ...options
      });
    } catch (error) {
      throw new Error(`Failed to call ${serviceName}: ${error.message}`);
    }
  }
}
```

## 6. Data Management Patterns

### Database per Service
```javascript
// User Service - PostgreSQL
class UserRepository {
  constructor(dbConnection) {
    this.db = dbConnection;
  }
  
  async createUser(userData) {
    const query = `
      INSERT INTO users (id, email, name, created_at)
      VALUES ($1, $2, $3, $4)
      RETURNING *
    `;
    
    const result = await this.db.query(query, [
      userData.id,
      userData.email,
      userData.name,
      new Date()
    ]);
    
    return result.rows[0];
  }
}

// Product Service - MongoDB  
class ProductRepository {
  constructor(mongoClient) {
    this.collection = mongoClient.db('products').collection('products');
  }
  
  async createProduct(productData) {
    const result = await this.collection.insertOne({
      ...productData,
      createdAt: new Date()
    });
    
    return { ...productData, _id: result.insertedId };
  }
  
  async findProductsByCategory(categoryId) {
    return await this.collection.find({ categoryId }).toArray();
  }
}
```

### Saga Pattern (Distributed Transactions)
```javascript
// Order Processing Saga
class OrderProcessingSaga {
  constructor(eventBus) {
    this.eventBus = eventBus;
    this.setupEventHandlers();
  }
  
  setupEventHandlers() {
    this.eventBus.on('order.created', this.handleOrderCreated.bind(this));
    this.eventBus.on('payment.completed', this.handlePaymentCompleted.bind(this));
    this.eventBus.on('payment.failed', this.handlePaymentFailed.bind(this));
    this.eventBus.on('inventory.reserved', this.handleInventoryReserved.bind(this));
    this.eventBus.on('inventory.reservation.failed', this.handleInventoryReservationFailed.bind(this));
  }
  
  async handleOrderCreated(event) {
    const { orderId, items, userId } = event.data;
    
    try {
      // Step 1: Reserve inventory
      await this.eventBus.emit('inventory.reserve', {
        orderId,
        items,
        sagaId: event.sagaId
      });
    } catch (error) {
      // Compensate - cancel order
      await this.eventBus.emit('order.cancel', {
        orderId,
        reason: 'Inventory reservation failed'
      });
    }
  }
  
  async handleInventoryReserved(event) {
    const { orderId, sagaId } = event.data;
    
    try {
      // Step 2: Process payment
      await this.eventBus.emit('payment.process', {
        orderId,
        sagaId
      });
    } catch (error) {
      // Compensate - release inventory
      await this.eventBus.emit('inventory.release', { orderId });
      await this.eventBus.emit('order.cancel', { orderId });
    }
  }
  
  async handlePaymentCompleted(event) {
    const { orderId, sagaId } = event.data;
    
    // Step 3: Confirm order
    await this.eventBus.emit('order.confirm', { orderId });
    await this.eventBus.emit('saga.completed', { sagaId });
  }
  
  async handlePaymentFailed(event) {
    const { orderId } = event.data;
    
    // Compensate - release inventory and cancel order
    await this.eventBus.emit('inventory.release', { orderId });
    await this.eventBus.emit('order.cancel', { orderId });
  }
}
```

### Event Sourcing
```javascript
class EventStore {
  constructor() {
    this.events = new Map(); // streamId -> events[]
  }
  
  async appendEvents(streamId, expectedVersion, events) {
    const stream = this.events.get(streamId) || [];
    
    if (expectedVersion !== -1 && stream.length !== expectedVersion) {
      throw new Error('Concurrency conflict');
    }
    
    const newEvents = events.map((event, index) => ({
      ...event,
      streamId,
      version: stream.length + index + 1,
      timestamp: new Date().toISOString()
    }));
    
    this.events.set(streamId, [...stream, ...newEvents]);
    
    // Publish events to event bus
    newEvents.forEach(event => {
      this.eventBus.publish(event.type, event);
    });
    
    return newEvents;
  }
  
  async getEvents(streamId, fromVersion = 0) {
    const stream = this.events.get(streamId) || [];
    return stream.filter(event => event.version > fromVersion);
  }
}

class OrderAggregate {
  constructor() {
    this.id = null;
    this.status = 'PENDING';
    this.items = [];
    this.version = 0;
  }
  
  static fromHistory(events) {
    const aggregate = new OrderAggregate();
    events.forEach(event => aggregate.apply(event));
    return aggregate;
  }
  
  createOrder(orderId, items, userId) {
    if (this.id) {
      throw new Error('Order already exists');
    }
    
    const event = {
      type: 'OrderCreated',
      data: { orderId, items, userId }
    };
    
    this.apply(event);
    return [event];
  }
  
  confirmOrder() {
    if (this.status !== 'PENDING') {
      throw new Error('Order cannot be confirmed');
    }
    
    const event = {
      type: 'OrderConfirmed',
      data: { orderId: this.id }
    };
    
    this.apply(event);
    return [event];
  }
  
  apply(event) {
    switch (event.type) {
      case 'OrderCreated':
        this.id = event.data.orderId;
        this.items = event.data.items;
        this.status = 'PENDING';
        break;
        
      case 'OrderConfirmed':
        this.status = 'CONFIRMED';
        break;
    }
    
    this.version++;
  }
}
```

## 7. Common Interview Questions

### 1. Monolith vs Microservices - Khi nào dùng gì?

**Microservices phù hợp khi:**
- Large team (>50 developers)
- Complex domain with clear boundaries
- Need independent scaling
- Different technology requirements
- High availability requirements

**Monolith phù hợp khi:**
- Small team (<10 developers)
- Simple domain
- Rapid prototyping
- Limited operational expertise
- Strong consistency requirements

### 2. Làm thế nào handle distributed transactions?

**Approaches:**
1. **Saga Pattern** - choreography hoặc orchestration
2. **Two-Phase Commit** - avoid nếu có thể
3. **Event Sourcing** - immutable event log
4. **Eventual Consistency** - accept temporary inconsistency

```javascript
// Saga Choreography Example
class PaymentService {
  async handleOrderCreated(event) {
    try {
      await this.processPayment(event.data);
      
      // Success - emit payment completed
      await this.eventBus.emit('payment.completed', {
        orderId: event.data.orderId,
        transactionId: result.transactionId
      });
    } catch (error) {
      // Failure - emit payment failed
      await this.eventBus.emit('payment.failed', {
        orderId: event.data.orderId,
        error: error.message
      });
    }
  }
}
```

### 3. Data consistency trong microservices?

**Consistency Patterns:**
- **Strong Consistency**: Synchronous calls, distributed transactions
- **Eventual Consistency**: Asynchronous events, compensating actions
- **Weak Consistency**: Best effort, eventual convergence

```javascript
// Eventual Consistency with Event-driven updates
class OrderService {
  async handleUserUpdated(event) {
    // Update denormalized user data in orders
    await this.orderRepository.updateUserInfo(
      event.data.userId,
      {
        userName: event.data.name,
        userEmail: event.data.email
      }
    );
  }
}
```

### 4. Service communication patterns - Sync vs Async?

**Synchronous (Request/Response):**
```javascript
// Good for: Real-time queries, immediate consistency needed
const user = await userService.getUser(userId);
const order = await orderService.createOrder({ userId, items });
```

**Asynchronous (Events):**
```javascript
// Good for: Fire-and-forget, eventual consistency, decoupling
eventBus.emit('user.created', { userId, email, name });
// Multiple services can react to this event independently
```

### 5. Cách handle service failures?

**Resilience Patterns:**
1. **Circuit Breaker** - prevent cascading failures
2. **Retry with Backoff** - handle transient failures
3. **Timeout** - prevent hanging requests
4. **Bulkhead** - isolate critical resources
5. **Fallback** - graceful degradation

```javascript
class ResilientServiceClient {
  async callWithResilience(operation) {
    const circuitBreaker = this.getCircuitBreaker();
    
    try {
      return await circuitBreaker.execute(async () => {
        return await this.retryWithBackoff(async () => {
          return await Promise.race([
            operation(),
            this.timeout(5000)
          ]);
        });
      });
    } catch (error) {
      return this.fallbackResponse(error);
    }
  }
  
  async retryWithBackoff(operation, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await operation();
      } catch (error) {
        if (attempt === maxRetries) throw error;
        
        const delay = Math.min(1000 * Math.pow(2, attempt - 1), 10000);
        await this.sleep(delay);
      }
    }
  }
  
  fallbackResponse(error) {
    return {
      error: 'Service temporarily unavailable',
      fallback: true
    };
  }
}
```

## 8. Deployment & Operations

### Docker Containerization
```dockerfile
# User Service Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

CMD ["npm", "start"]
```

### Kubernetes Deployment
```yaml
# user-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: user-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

### Service Mesh (Istio)
```yaml
# user-service-virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: user-service
spec:
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: user-service
        subset: canary
      weight: 100
  - route:
    - destination:
        host: user-service
        subset: stable
      weight: 100
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
  - timeout: 10s
  retries:
    attempts: 3
    perTryTimeout: 2s
```

### Monitoring & Observability
```javascript
// Distributed Tracing
const opentelemetry = require('@opentelemetry/api');
const { NodeSDK } = require('@opentelemetry/auto-instrumentations-node');

class UserService {
  async createUser(userData) {
    const span = opentelemetry.trace.getActiveSpan();
    span?.setAttributes({
      'user.email': userData.email,
      'service.name': 'user-service'
    });
    
    try {
      const user = await this.userRepository.create(userData);
      
      span?.addEvent('user.created', {
        'user.id': user.id
      });
      
      return user;
    } catch (error) {
      span?.recordException(error);
      span?.setStatus({
        code: opentelemetry.SpanStatusCode.ERROR,
        message: error.message
      });
      throw error;
    }
  }
}

// Metrics Collection
const prometheus = require('prom-client');

const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code', 'service']
});

const businessMetrics = new prometheus.Counter({
  name: 'users_created_total',
  help: 'Total number of users created',
  labelNames: ['service']
});

class MetricsMiddleware {
  static trackRequest() {
    return (req, res, next) => {
      const start = Date.now();
      
      res.on('finish', () => {
        const duration = (Date.now() - start) / 1000;
        
        httpRequestDuration
          .labels(req.method, req.route?.path || req.path, res.statusCode, 'user-service')
          .observe(duration);
      });
      
      next();
    };
  }
}
```

## 9. Best Practices & Anti-patterns

### Best Practices
1. **Start with Monolith** - migrate to microservices when needed
2. **Design for Failure** - assume services will fail
3. **Automate Everything** - CI/CD, testing, deployment
4. **Monitor Everything** - metrics, logs, traces
5. **Secure by Default** - authentication, authorization, encryption

### Common Anti-patterns
1. **Distributed Monolith** - services too tightly coupled
2. **Chatty Services** - too many synchronous calls
3. **Shared Database** - violates service autonomy
4. **Synchronous Communication Everywhere** - creates tight coupling
5. **No Monitoring** - can't troubleshoot distributed issues

### Testing Strategies
```javascript
// Contract Testing with Pact
const { Pact } = require('@pact-foundation/pact');

describe('User Service Consumer', () => {
  const provider = new Pact({
    consumer: 'Order Service',
    provider: 'User Service'
  });
  
  it('should get user by id', async () => {
    await provider
      .given('user exists')
      .uponReceiving('a request for user')
      .withRequest({
        method: 'GET',
        path: '/users/123',
        headers: {
          'Accept': 'application/json'
        }
      })
      .willRespondWith({
        status: 200,
        headers: {
          'Content-Type': 'application/json'
        },
        body: {
          id: '123',
          name: 'John Doe',
          email: 'john@example.com'
        }
      });
      
    const response = await userServiceClient.getUser('123');
    expect(response.name).toBe('John Doe');
  });
});

// Integration Testing
class ServiceIntegrationTest {
  async setupTestEnvironment() {
    // Start test containers
    this.userServiceContainer = await this.testContainers.start('user-service');
    this.orderServiceContainer = await this.testContainers.start('order-service');
    this.databaseContainer = await this.testContainers.start('postgres');
    
    // Wait for services to be ready
    await this.waitForServiceHealth(this.userServiceContainer);
    await this.waitForServiceHealth(this.orderServiceContainer);
  }
  
  async testOrderCreationFlow() {
    // Create user
    const user = await this.userServiceClient.createUser({
      name: 'Test User',
      email: 'test@example.com'
    });
    
    // Create order
    const order = await this.orderServiceClient.createOrder({
      userId: user.id,
      items: [{ productId: '1', quantity: 2 }]
    });
    
    expect(order.userId).toBe(user.id);
    expect(order.status).toBe('CREATED');
  }
}
