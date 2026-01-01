# Microservices & DevOps Interview Questions

## 1. Microservices Architecture
**Concept:** Breaking down a large monolithic application into smaller, independent services that communicate via APIs.

### What are Microservices?
A software architectural style where an application is built as a collection of small, autonomous services that:
- Run in their own process
- Communicate via lightweight protocols (HTTP/REST, gRPC, Message Queues)
- Are independently deployable
- Are organized around business capabilities
- Can be written in different programming languages

### Monolithic vs Microservices:

| Aspect | Monolithic | Microservices |
|--------|-----------|---------------|
| **Structure** | Single codebase | Multiple independent services |
| **Deployment** | Deploy entire app | Deploy individual services |
| **Scaling** | Scale entire app | Scale individual services |
| **Technology** | One tech stack | Multiple tech stacks possible |
| **Development** | Harder to work in parallel | Teams work independently |
| **Testing** | Test entire app | Test individual services |
| **Failure** | One failure = entire app down | Isolated failures |

**Example - E-Commerce Application:**

**Monolithic:**
```
┌─────────────────────────────────────┐
│       E-Commerce Application        │
│  ┌────────────────────────────────┐ │
│  │  User Management               │ │
│  │  Product Catalog               │ │
│  │  Shopping Cart                 │ │
│  │  Order Management              │ │
│  │  Payment Processing            │ │
│  │  Inventory Management          │ │
│  │  Notification Service          │ │
│  └────────────────────────────────┘ │
│        Single Database              │
└─────────────────────────────────────┘
```

**Microservices:**
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   User       │  │   Product    │  │   Order      │
│   Service    │  │   Service    │  │   Service    │
│   (Node.js)  │  │   (Java)     │  │   (Python)   │
│   Port: 3001 │  │   Port: 8080 │  │   Port: 5000 │
└──────────────┘  └──────────────┘  └──────────────┘
       │                 │                  │
    ┌──▼─────┐      ┌───▼────┐       ┌────▼───┐
    │User DB │      │Prod DB │       │Order DB│
    └────────┘      └────────┘       └────────┘

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Payment    │  │  Inventory   │  │ Notification │
│   Service    │  │   Service    │  │   Service    │
│   (C#)       │  │   (Go)       │  │   (Node.js)  │
│   Port: 9000 │  │   Port: 7000 │  │   Port: 4000 │
└──────────────┘  └──────────────┘  └──────────────┘
       │                 │                  │
   ┌───▼────┐      ┌─────▼──┐        ┌─────▼───┐
   │Pay DB  │      │Inv DB  │        │Queue    │
   └────────┘      └────────┘        └─────────┘
```

### Microservices Communication Patterns:

**1. Synchronous Communication (HTTP/REST):**
```javascript
// Order Service calling Payment Service
const axios = require('axios');

async function createOrder(orderData) {
  try {
    // 1. Create order
    const order = await saveOrder(orderData);
    
    // 2. Call Payment Service
    const paymentResponse = await axios.post('http://payment-service:9000/api/payments', {
      orderId: order.id,
      amount: order.total,
      customerId: order.customerId
    });
    
    // 3. Update order with payment status
    order.paymentId = paymentResponse.data.id;
    order.status = 'PAID';
    await updateOrder(order);
    
    return order;
  } catch (error) {
    console.error('Order creation failed:', error);
    throw error;
  }
}
```

**2. Asynchronous Communication (Message Queue):**
```javascript
// Using RabbitMQ / AWS SQS / Kafka
const amqp = require('amqplib');

// Order Service - Publish event
async function publishOrderCreatedEvent(order) {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  
  const exchange = 'order-events';
  await channel.assertExchange(exchange, 'fanout', { durable: true });
  
  const message = {
    eventType: 'ORDER_CREATED',
    orderId: order.id,
    customerId: order.customerId,
    items: order.items,
    total: order.total,
    timestamp: new Date()
  };
  
  channel.publish(exchange, '', Buffer.from(JSON.stringify(message)));
  console.log('Order created event published');
}

// Inventory Service - Subscribe to event
async function subscribeToOrderEvents() {
  const connection = await amqp.connect('amqp://localhost');
  const channel = await connection.createChannel();
  
  const exchange = 'order-events';
  await channel.assertExchange(exchange, 'fanout', { durable: true });
  
  const queue = await channel.assertQueue('inventory-queue', { durable: true });
  await channel.bindQueue(queue.queue, exchange, '');
  
  channel.consume(queue.queue, async (msg) => {
    const event = JSON.parse(msg.content.toString());
    
    if (event.eventType === 'ORDER_CREATED') {
      // Update inventory
      await reduceInventory(event.items);
      console.log('Inventory updated for order:', event.orderId);
    }
    
    channel.ack(msg);
  });
}
```

### Advantages of Microservices:
- ✅ **Independent Deployment** - Deploy one service without affecting others
- ✅ **Technology Diversity** - Use best tool for each job
- ✅ **Scalability** - Scale only the services that need it
- ✅ **Fault Isolation** - Failure in one service doesn't crash entire app
- ✅ **Faster Development** - Teams work independently
- ✅ **Easier Maintenance** - Smaller codebases are easier to understand

### Challenges of Microservices:
- ❌ **Distributed System Complexity** - Network calls, latency
- ❌ **Data Consistency** - Managing transactions across services
- ❌ **Testing Complexity** - Need to test service interactions
- ❌ **Deployment Overhead** - More services to deploy and monitor
- ❌ **Monitoring & Debugging** - Distributed tracing needed

---

## 2. Circuit Breaker Pattern
**Concept:** A design pattern that prevents cascading failures in microservices by stopping calls to failing services.

### Problem Without Circuit Breaker:
```
User Request → Service A → Service B (Down) ❌
                   ↓
            Keeps trying...
            Timeout... Timeout... Timeout...
                   ↓
            Service A overloaded 💥
            Entire system slows down 🐌
```

### Solution With Circuit Breaker:
```
User Request → Service A → Circuit Breaker → Service B (Down) ❌
                              ↓
                    Detects failure pattern
                         Opens circuit
                              ↓
                    Returns fallback response
                    Service A stays healthy ✅
```

### Circuit Breaker States:

```
                  ┌──────────┐
         Success  │          │  Failure threshold
        ┌─────────│  CLOSED  │────────────────┐
        │         │          │                │
        │         └──────────┘                ▼
        │                               ┌──────────┐
        │                               │          │
        │                               │   OPEN   │
        │                               │          │
        │                               └──────────┘
        │                                     │
        │         ┌──────────┐               │ Timeout
        │         │          │               │
        └─────────│HALF-OPEN │◄──────────────┘
        Success   │          │
                  └──────────┘
                       │
                Failure│
                       ▼
                  ┌──────────┐
                  │   OPEN   │
                  └──────────┘
```

**States Explained:**
1. **CLOSED** (Normal) - Requests pass through normally. If failures exceed threshold, open circuit.
2. **OPEN** (Failing) - All requests fail fast without calling service. After timeout, move to HALF-OPEN.
3. **HALF-OPEN** (Testing) - Allow limited requests through to test if service recovered. If success, close circuit. If failure, open again.

### Implementation Example (Node.js):

**Using `opossum` library:**
```javascript
const CircuitBreaker = require('opossum');
const axios = require('axios');

// Function to call external service
async function callPaymentService(paymentData) {
  const response = await axios.post('http://payment-service:9000/api/pay', paymentData);
  return response.data;
}

// Circuit breaker options
const options = {
  timeout: 3000,                  // 3 seconds timeout
  errorThresholdPercentage: 50,   // Open circuit if 50% requests fail
  resetTimeout: 10000,            // Try again after 10 seconds (HALF-OPEN)
  rollingCountTimeout: 10000,     // Time window for counting errors
  rollingCountBuckets: 10,        // Number of buckets in time window
  name: 'PaymentServiceBreaker'
};

// Create circuit breaker
const breaker = new CircuitBreaker(callPaymentService, options);

// Fallback function when circuit is open
breaker.fallback((paymentData) => {
  console.log('Circuit is OPEN, using fallback');
  return {
    status: 'PENDING',
    message: 'Payment service unavailable. Will process later.'
  };
});

// Event listeners
breaker.on('open', () => {
  console.log('Circuit breaker is OPEN - stopping requests');
});

breaker.on('halfOpen', () => {
  console.log('Circuit breaker is HALF-OPEN - testing service');
});

breaker.on('close', () => {
  console.log('Circuit breaker is CLOSED - service recovered');
});

breaker.on('fallback', (result) => {
  console.log('Fallback executed:', result);
});

// Usage in your service
async function processPayment(paymentData) {
  try {
    const result = await breaker.fire(paymentData);
    return result;
  } catch (error) {
    console.error('Payment failed:', error);
    throw error;
  }
}

// Express endpoint
app.post('/api/orders', async (req, res) => {
  try {
    const paymentResult = await processPayment(req.body.payment);
    res.json({ success: true, payment: paymentResult });
  } catch (error) {
    res.status(500).json({ error: 'Payment processing failed' });
  }
});
```

**Using `resilience4js` (More features):**
```javascript
const { CircuitBreaker } = require('resilience4js');

const breaker = new CircuitBreaker({
  failureRateThreshold: 50,           // 50% failure rate
  waitDurationInOpenState: 10000,     // 10 seconds
  permittedNumberOfCallsInHalfOpenState: 3,  // Test with 3 calls
  slidingWindowSize: 10,              // Look at last 10 calls
  minimumNumberOfCalls: 5             // Need at least 5 calls before calculating
});

breaker.execute(
  () => callPaymentService(paymentData),
  (error) => {
    // Fallback
    return { status: 'QUEUED', message: 'Will retry later' };
  }
);
```

### Spring Boot Implementation (Java):

**Using Resilience4j:**
```java
import io.github.resilience4j.circuitbreaker.annotation.CircuitBreaker;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class PaymentService {
    
    private final RestTemplate restTemplate;
    
    public PaymentService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        String url = "http://payment-service:9000/api/pay";
        return restTemplate.postForObject(url, request, PaymentResponse.class);
    }
    
    // Fallback method
    public PaymentResponse paymentFallback(PaymentRequest request, Exception ex) {
        System.out.println("Circuit breaker activated. Using fallback.");
        return new PaymentResponse("PENDING", "Service temporarily unavailable");
    }
}
```

**Configuration (application.yml):**
```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        registerHealthIndicator: true
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        permittedNumberOfCallsInHalfOpenState: 3
        waitDurationInOpenState: 10s
        failureRateThreshold: 50
        slowCallRateThreshold: 50
        slowCallDurationThreshold: 2s
```

### Real-World Example - E-Commerce Checkout:

```javascript
// Order Service with Circuit Breaker for multiple services
const CircuitBreaker = require('opossum');

// Circuit breakers for each dependency
const paymentBreaker = new CircuitBreaker(callPaymentService, { timeout: 3000 });
const inventoryBreaker = new CircuitBreaker(callInventoryService, { timeout: 2000 });
const notificationBreaker = new CircuitBreaker(sendNotification, { timeout: 1000 });

// Fallbacks
paymentBreaker.fallback(() => ({ status: 'QUEUED', message: 'Payment queued' }));
inventoryBreaker.fallback(() => ({ reserved: false, message: 'Will check later' }));
notificationBreaker.fallback(() => ({ sent: false })); // Non-critical, ignore failure

async function createOrder(orderData) {
  try {
    // 1. Reserve inventory (critical)
    const inventory = await inventoryBreaker.fire(orderData.items);
    if (!inventory.reserved) {
      return { error: 'Inventory not available' };
    }
    
    // 2. Process payment (critical)
    const payment = await paymentBreaker.fire(orderData.payment);
    
    // 3. Send notification (non-critical, can fail)
    notificationBreaker.fire({
      email: orderData.email,
      message: 'Order confirmed'
    }).catch(() => console.log('Notification failed, will retry'));
    
    return {
      orderId: generateId(),
      status: payment.status,
      items: orderData.items
    };
  } catch (error) {
    console.error('Order creation failed:', error);
    throw error;
  }
}
```

### Benefits of Circuit Breaker:
- ✅ **Prevents cascading failures** - Stops calling failing services
- ✅ **Fail fast** - Returns error immediately instead of waiting for timeout
- ✅ **Self-healing** - Automatically retries after timeout
- ✅ **Resource protection** - Prevents thread/connection exhaustion
- ✅ **Graceful degradation** - Fallback responses keep system partially functional

---

## 3. CI/CD Pipeline Setup
**Concept:** Continuous Integration and Continuous Deployment - automating the software delivery process from code commit to production.

### CI/CD Pipeline Stages:

```
Developer commits code
        ↓
   ┌─────────────────────────────────────────────┐
   │           CI (Continuous Integration)       │
   ├─────────────────────────────────────────────┤
   │  1. Source Control (Git)                    │
   │  2. Build & Compile                         │
   │  3. Unit Tests                              │
   │  4. Code Quality Analysis (SonarQube)       │
   │  5. Security Scan (OWASP, Snyk)            │
   │  6. Package (Docker Image)                  │
   └─────────────────────────────────────────────┘
        ↓
   ┌─────────────────────────────────────────────┐
   │           CD (Continuous Deployment)        │
   ├─────────────────────────────────────────────┤
   │  7. Deploy to DEV Environment               │
   │  8. Integration Tests                       │
   │  9. Deploy to STAGING                       │
   │ 10. Smoke Tests                             │
   │ 11. Manual Approval (Optional)              │
   │ 12. Deploy to PRODUCTION                    │
   │ 13. Health Checks & Monitoring              │
   └─────────────────────────────────────────────┘
        ↓
   Production Ready ✅
```

### Example 1: GitHub Actions Pipeline

**`.github/workflows/ci-cd.yml`:**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '18.x'
  DOCKER_REGISTRY: docker.io
  IMAGE_NAME: myapp

jobs:
  # Job 1: Build and Test
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
      
      - name: Build application
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-files
          path: dist/
  
  # Job 2: Security Scan
  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Run Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: OWASP Dependency Check
        run: |
          npm audit --audit-level=moderate
  
  # Job 3: Build Docker Image
  build-docker:
    runs-on: ubuntu-latest
    needs: [build-and-test, security-scan]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  # Job 4: Deploy to DEV
  deploy-dev:
    runs-on: ubuntu-latest
    needs: build-docker
    environment: development
    
    steps:
      - name: Deploy to DEV (AWS ECS)
        run: |
          aws ecs update-service \
            --cluster dev-cluster \
            --service myapp-service \
            --force-new-deployment \
            --region us-east-1
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster dev-cluster \
            --services myapp-service \
            --region us-east-1
      
      - name: Run integration tests
        run: |
          npm run test:integration
        env:
          API_URL: https://dev.myapp.com
  
  # Job 5: Deploy to STAGING
  deploy-staging:
    runs-on: ubuntu-latest
    needs: deploy-dev
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
      - name: Deploy to STAGING
        run: |
          kubectl set image deployment/myapp-deployment \
            myapp=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n staging
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/myapp-deployment -n staging
      
      - name: Run smoke tests
        run: |
          curl -f https://staging.myapp.com/health || exit 1
  
  # Job 6: Deploy to PRODUCTION (Manual Approval Required)
  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - name: Deploy to PRODUCTION
        run: |
          kubectl set image deployment/myapp-deployment \
            myapp=${{ env.DOCKER_REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
            -n production
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/myapp-deployment -n production
      
      - name: Health check
        run: |
          curl -f https://myapp.com/health || exit 1
      
      - name: Notify deployment
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Deployment to PRODUCTION completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Example 2: Jenkins Pipeline

**`Jenkinsfile`:**
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        IMAGE_NAME = 'myapp'
        AWS_REGION = 'us-east-1'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/myorg/myapp.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Lint & Test') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
                stage('Unit Tests') {
                    steps {
                        sh 'npm test -- --coverage'
                        publishHTML(target: [
                            reportDir: 'coverage',
                            reportFiles: 'index.html',
                            reportName: 'Coverage Report'
                        ])
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh 'npm audit --audit-level=moderate'
                // Snyk scan
                snykSecurity(
                    snykInstallation: 'Snyk',
                    snykTokenId: 'snyk-token',
                    severity: 'high'
                )
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                        def image = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to DEV') {
            steps {
                sh '''
                    aws ecs update-service \
                        --cluster dev-cluster \
                        --service myapp-service \
                        --force-new-deployment \
                        --region ${AWS_REGION}
                '''
            }
        }
        
        stage('Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }
        
        stage('Deploy to STAGING') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    kubectl set image deployment/myapp \
                        myapp=${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n staging
                    kubectl rollout status deployment/myapp -n staging
                '''
            }
        }
        
        stage('Smoke Tests') {
            steps {
                sh 'curl -f https://staging.myapp.com/health'
            }
        }
        
        stage('Approval') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
            }
        }
        
        stage('Deploy to PRODUCTION') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                    kubectl set image deployment/myapp \
                        myapp=${DOCKER_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n production
                    kubectl rollout status deployment/myapp -n production
                '''
            }
        }
    }
    
    post {
        success {
            slackSend(
                color: 'good',
                message: "Deployment successful: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "Deployment failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

### Example 3: GitLab CI/CD

**`.gitlab-ci.yml`:**
```yaml
stages:
  - build
  - test
  - security
  - package
  - deploy-dev
  - deploy-staging
  - deploy-production

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_NAME: myapp

# Build Stage
build:
  stage: build
  image: node:18
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  cache:
    paths:
      - node_modules/

# Test Stage
test:unit:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run test -- --coverage
  coverage: '/Statements\s*:\s*([^%]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:lint:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run lint

# Security Stage
security:dependency-scan:
  stage: security
  image: node:18
  script:
    - npm audit --audit-level=moderate
  allow_failure: true

security:sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml

# Package Stage
package:docker:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest

# Deploy to DEV
deploy:dev:
  stage: deploy-dev
  image: alpine/k8s:latest
  script:
    - kubectl config use-context dev-cluster
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n dev
    - kubectl rollout status deployment/myapp -n dev
  environment:
    name: development
    url: https://dev.myapp.com

# Deploy to STAGING
deploy:staging:
  stage: deploy-staging
  image: alpine/k8s:latest
  script:
    - kubectl config use-context staging-cluster
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n staging
    - kubectl rollout status deployment/myapp -n staging
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - main

# Deploy to PRODUCTION (Manual)
deploy:production:
  stage: deploy-production
  image: alpine/k8s:latest
  script:
    - kubectl config use-context prod-cluster
    - kubectl set image deployment/myapp myapp=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA -n production
    - kubectl rollout status deployment/myapp -n production
  environment:
    name: production
    url: https://myapp.com
  when: manual
  only:
    - main
```

### CI/CD Best Practices:
- ✅ **Automated Testing** - Run tests on every commit
- ✅ **Small, Frequent Commits** - Easier to identify issues
- ✅ **Fast Feedback** - Keep pipeline fast (<10 minutes)
- ✅ **Parallel Execution** - Run independent stages in parallel
- ✅ **Environment Parity** - DEV/STAGING/PROD should be similar
- ✅ **Rollback Strategy** - Quick rollback on failures
- ✅ **Monitoring & Alerts** - Monitor deployments in real-time
- ✅ **Infrastructure as Code** - Version control infrastructure
- ✅ **Security Scans** - Scan for vulnerabilities early
- ✅ **Manual Approval for Production** - Add human checkpoint

---

## Summary Table

| Topic | Key Takeaway |
|-------|-------------|
| **Microservices** | Break monolith into independent services; scalable, technology-diverse |
| **Circuit Breaker** | Prevent cascading failures; fail fast, self-healing |
| **CI/CD Pipeline** | Automate build, test, deploy; faster delivery, fewer errors |
| **Communication** | Sync (REST) for simple calls; Async (Queue) for decoupling |
| **Best Practices** | Monitor everything, automate testing, use Docker/K8s |
