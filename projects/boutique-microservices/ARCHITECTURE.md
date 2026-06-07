# System Architecture Diagrams

## 1. High-Level System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                               │
│                             BOUTIQUE SYSTEM OVERVIEW                         │
│                                                                               │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌──────────────────┐
                              │   Web Browser    │
                              │  (User's Device) │
                              └────────┬─────────┘
                                       │ HTTPS
                                       ▼
                    ┌──────────────────────────────────┐
                    │   React Frontend                 │
                    │   (Port 3000)                    │
                    │  - Product Browse                │
                    │  - Authentication UI             │
                    │  - Shopping Cart                 │
                    │  - Order Management              │
                    └──────────┬──────────────────────┘
                               │ HTTP/REST API Calls
                               ▼
                    ┌──────────────────────────────────┐
                    │     API GATEWAY                  │
                    │     (Port 3001)                  │
                    │  - Request Routing               │
                    │  - Load Balancing                │
                    │  - Security Headers              │
                    │  - Metrics Collection            │
                    └───┬──────┬──────┬───────┬────────┘
                        │      │      │       │
        ┌───────────────┘      │      │       └───────────────┐
        │                      │      │                       │
        ▼                      ▼      ▼                       ▼
    ┌────────┐            ┌────────┐ ┌─────────┐        ┌────────┐
    │ Auth   │            │Product │ │ Order   │        │ User   │
    │Service │            │Service │ │ Service │        │Service │
    │(3002)  │            │(3003)  │ └─────────┘        │(3006)  │
    └────┬───┘            └────┬───┘      │              └────┬───┘
         │                     │          ▼
         │                     │      ┌────────┐
         │                     │      │ Order  │
         │                     │      │ History│
         │                     │      │(3005)  │
         │                     │      └────┬───┘
         │                     │            │
         └─────────┬───────────┴────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │   PostgreSQL         │
        │   (Port 5432)        │
        │  ┌──────────────────┐│
        │  │ auth_db          ││  Users, Sessions
        │  ├──────────────────┤│
        │  │ products_db      ││  Products, Images
        │  ├──────────────────┤│
        │  │ orders_db        ││  Orders, Items
        │  ├──────────────────┤│
        │  │ users_db         ││  Profiles, Addresses
        │  ├──────────────────┤│
        │  │ boutique_db      ││  Demo Data
        │  └──────────────────┘│
        └──────────────────────┘

        ┌──────────────────────┐
        │  MONITORING STACK    │
        ├──────────────────────┤
        │ Prometheus (9090)    │  ← Scrapes all services every 15s
        ├──────────────────────┤
        │ Grafana (3007)       │  ← Visualizes metrics
        └──────────────────────┘
```

---

## 2. Request Flow Diagram

### User Authentication Flow

```
┌─────────────┐
│   Browser   │
│  (Frontend) │
└──────┬──────┘
       │ 1. User clicks "Login"
       │
       ▼
┌──────────────────────────┐
│ React Login Component    │
│ - Render login form      │
│ - Validate input         │
└──────┬───────────────────┘
       │ 2. POST /api/auth/login
       │    {email, password}
       │
       ▼
┌──────────────────────────┐
│   API Gateway            │
│ - Validate request       │
│ - Record metrics         │
│ - Forward to Auth Service│
└──────┬───────────────────┘
       │ 3. Route to /auth/login
       │
       ▼
┌──────────────────────────┐
│   Auth Service (3002)    │
│ - Hash password          │
│ - Query users table      │
│ - Verify credentials     │
└──────┬───────────────────┘
       │ 4. Query: SELECT * FROM users WHERE email = ?
       │
       ▼
┌──────────────────────────┐
│  PostgreSQL (auth_db)    │
│ - Find user record       │
│ - Return user object     │
└──────┬───────────────────┘
       │ 5. Return user data
       │
       ▼
┌──────────────────────────┐
│   Auth Service           │
│ - Generate JWT token     │
│ - Sign with secret       │
│ - Return token           │
└──────┬───────────────────┘
       │ 6. {token, user_id, expires_in}
       │
       ▼
┌──────────────────────────┐
│   API Gateway            │
│ - Record success metric  │
│ - Return response        │
└──────┬───────────────────┘
       │ 7. HTTP 200 + JWT token
       │
       ▼
┌──────────────────────────┐
│   Browser                │
│ - Save JWT to localStorage
│ - Redirect to dashboard  │
└──────────────────────────┘
```

### Product Purchase Flow

```
Frontend                    Gateway                Services              Database
─────────                   ───────                ────────              ────────

User adds 
product to 
cart
   │
   ├─────POST /api/cart/add──────→
   │                               │
   │                               ├─→ validate JWT
   │                               │
   │                               ├─→ call Product Service
   │                               │      │
   │                               │      └─→ Check inventory
   │                               │          (products_db)
   │                               │      ↓
   │                               │      SELECT stock FROM products
   │                               │                                  ↓
   │                               │                          Return stock level
   │                               │      ↑
   │                               ├──────┘ Success: 5 units in stock
   │                               │
   │                               ├─→ Add to cart (orders_db)
   │                               │      │
   │                               │      └─→ INSERT INTO order_items
   │                               │                                  ↓
   │                               │                          Return: item added
   │◄─────200 OK + cart────────────┤
   │

User clicks
Checkout
   │
   ├─────POST /api/orders/create──→
   │                               │
   │                               ├─→ Validate user JWT
   │                               │
   │                               ├─→ Call Order Service
   │                               │      │
   │                               │      ├─→ Verify user (Auth Service)
   │                               │      │      │ (3002)
   │                               │      │      └─→ Validate JWT
   │                               │      │
   │                               │      ├─→ Verify inventory (Product Service)
   │                               │      │      │ (3003)
   │                               │      │      └─→ Check stock levels
   │                               │      │
   │                               │      ├─→ Create order (orders_db)
   │                               │      │      │
   │                               │      │      ├─→ INSERT INTO orders
   │                               │      │      └─→ UPDATE products SET stock = stock - qty
   │                               │      │
   │                               │      └─→ Return: Order #12345
   │                               │
   │◄─────200 OK + Order ID────────┤
   │
   │
   └─→ Redirect to /orders/12345
```

---

## 3. Container Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                   Docker Host                                 │
│                                                                │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              boutique-network (bridge)                 │  │
│  │                                                         │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐   │  │
│  │  │  Frontend    │  │  Gateway     │  │   Auth     │   │  │
│  │  │  Port 3000   │  │  Port 3001   │  │  Port 3002 │   │  │
│  │  │              │  │              │  │            │   │  │
│  │  │  Node.js 20  │  │  Node.js 20  │  │ Node.js 20 │   │  │
│  │  │  + Serve     │  │  + Express   │  │ + Express  │   │  │
│  │  │  + React     │  │  + Proxy MW  │  │ + bcryptjs │   │  │
│  │  └──────────────┘  └──────────────┘  └────────────┘   │  │
│  │                                                         │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐   │  │
│  │  │   Products   │  │    Orders    │  │   Users    │   │  │
│  │  │  Port 3003   │  │  Port 3005   │  │ Port 3006  │   │  │
│  │  │              │  │              │  │            │   │  │
│  │  │ Node.js 20   │  │ Node.js 20   │  │Node.js 20  │   │  │
│  │  │ + Express    │  │ + Express    │  │+ Express   │   │  │
│  │  │ + Sharp      │  │              │  │            │   │  │
│  │  └──────────────┘  └──────────────┘  └────────────┘   │  │
│  │                                                         │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐   │  │
│  │  │ PostgreSQL   │  │ Prometheus   │  │  Grafana   │   │  │
│  │  │ Port 5432    │  │  Port 9090   │  │ Port 3007  │   │  │
│  │  │              │  │              │  │            │   │  │
│  │  │ Postgres 15  │  │ Prometheus   │  │ Grafana 10 │   │  │
│  │  │ Alpine       │  │ Latest       │  │ Latest     │   │  │
│  │  └──────────────┘  └──────────────┘  └────────────┘   │  │
│  │       ▲              ▲                                  │  │
│  │       │              │                                  │  │
│  │   (All services connected via network)                │  │
│  │                                                         │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                                │
│  ┌────────────────────────────────────────────────────────┐  │
│  │            Volumes (Data Persistence)                  │  │
│  │  ┌────────────────────────────────────────────────┐   │  │
│  │  │ postgres_data: /var/lib/postgresql/data        │   │  │
│  │  │ prometheus_data: /prometheus                   │   │  │
│  │  │ grafana_data: /var/lib/grafana                 │   │  │
│  │  └────────────────────────────────────────────────┘   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                                │
│  ┌────────────────────────────────────────────────────────┐  │
│  │      Port Mappings (Host : Container)                 │  │
│  │                                                         │  │
│  │  3000 → 3000 (Frontend)                               │  │
│  │  3001 → 3001 (Gateway)                                │  │
│  │  3002 → 3002 (Auth)                                   │  │
│  │  3003 → 3003 (Products)                               │  │
│  │  3004 → 3004 (Orders)                                 │  │
│  │  3005 → 3005 (Order History)                          │  │
│  │  3006 → 3006 (Users)                                  │  │
│  │  5432 → 5432 (PostgreSQL)                             │  │
│  │  9090 → 9090 (Prometheus)                             │  │
│  │  3007 → 3000 (Grafana)                                │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Data Flow Architecture

```
                    ┌─────────────────┐
                    │   End User      │
                    │   (Browser)     │
                    └────────┬────────┘
                             │
                    HTTP/REST │ Requests
                             ▼
                    ┌─────────────────┐
                    │  React Frontend │
                    │   (Port 3000)   │
                    └────────┬────────┘
                             │
                             │ HTTPS/JSON
                             │
                    ┌────────▼────────┐
                    │  Load Balancer  │
                    │   (ALB - AWS)   │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  API Gateway    │
                    │   (Port 3001)   │
                    └────────┬────────┘
                             │
             ┌───────────────┼───────────────┐
             │               │               │
        ┌────▼────┐   ┌─────▼─────┐   ┌────▼────┐
        │Auth     │   │ Product   │   │Orders   │
        │Service  │   │ Service   │   │Service  │
        │(3002)   │   │ (3003)    │   │(3005)   │
        └────┬────┘   └─────┬─────┘   └────┬────┘
             │               │               │
             │    ┌──────────▼──────────┐    │
             │    │   PostgreSQL        │    │
             │    │   (Port 5432)       │    │
             │    │  ┌────────────────┐ │    │
             │    │  │auth_db         │ │◄───┘
             │    │  │products_db     │ │◄────┐
             │    │  │orders_db       │ │    │
             │    │  │users_db        │ │    │
             │    │  └────────────────┘ │    │
             │    └─────────────────────┘    │
             │                               │
             └──────────────────────────────┘

             ┌──────────────────────────────┐
             │    Monitoring Pipeline       │
             │                              │
             │  Every 15 seconds:           │
             │  ┌────────────────────────┐ │
             │  │ Prometheus scrapes     │ │
             │  │ /metrics on each svc   │ │
             │  └───────┬────────────────┘ │
             │          │                   │
             │  ┌───────▼────────────────┐ │
             │  │ Time-series database   │ │
             │  │ (Prometheus storage)   │ │
             │  └───────┬────────────────┘ │
             │          │                   │
             │  ┌───────▼────────────────┐ │
             │  │ Grafana dashboards     │ │
             │  │ Real-time visualization│ │
             │  └────────────────────────┘ │
             └──────────────────────────────┘
```

---

## 5. Service Communication Matrix

```
                    ┌──────────────────────────────────────────────┐
                    │         Service Interaction Matrix           │
                    └──────────────────────────────────────────────┘

                  To: ↓ From: →  Gateway  Auth   Product Order  Orders User  DB
                  ──────────────────────────────────────────────────────────────
                       Gateway      -     ✓✓      ✓✓      ✓✓     ✓✓     ✓✓   
                       Auth         ✓     -       ─       ─      ─      ─     ✓
                       Product      ✓     ✓       -       ✓      ─      ─     ✓
                       Order        ✓     ✓✓      ✓✓      -      ─      ─     ✓
                       Orders       ✓     ─       ─       ─      -      ─     ✓
                       User         ✓     ─       ─       ─      ✓✓     -     ✓
                       DB           ✓     ✓       ✓       ✓      ✓      ✓     -

Legend:
✓✓ = Strong dependency (calls in request flow)
✓  = Weak dependency (optional or admin only)
─  = No direct communication
```

**Request Flow Example (Add to Cart):**
```
User → Frontend
     → Gateway (API call)
     → Order Service (calls)
        ├→ Auth Service (validate JWT)
        │  └→ PostgreSQL (auth_db) - verify user
        └→ Product Service (check inventory)
           └→ PostgreSQL (products_db) - verify stock
     ← Return: Item added to cart
```

---

## 6. Database Schema Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Architecture                        │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────┐
│      auth_db         │  Owner: Auth Service
├──────────────────────┤
│ users                │
│  ├─ id (UUID PK)     │
│  ├─ email (UNIQUE)   │
│  ├─ password_hash    │
│  └─ created_at       │
│                      │
│ sessions             │
│  ├─ id (UUID PK)     │
│  ├─ user_id (FK)     │
│  ├─ token            │
│  └─ expires_at       │
└──────────────────────┘

┌──────────────────────┐
│    products_db       │  Owner: Product Service
├──────────────────────┤
│ products             │
│  ├─ id (UUID PK)     │
│  ├─ name             │
│  ├─ price            │
│  ├─ description      │
│  └─ inventory        │
│                      │
│ product_images       │
│  ├─ id (UUID PK)     │
│  ├─ product_id (FK)  │
│  └─ url              │
│                      │
│ product_variants     │
│  ├─ id (UUID PK)     │
│  ├─ product_id (FK)  │
│  ├─ size             │
│  ├─ color            │
│  └─ stock            │
└──────────────────────┘

┌──────────────────────┐
│    orders_db         │  Owners: Order Service
├──────────────────────┤    & Orders Service
│ orders               │
│  ├─ id (UUID PK)     │
│  ├─ user_id (FK)     │
│  ├─ total            │
│  ├─ status           │
│  └─ created_at       │
│                      │
│ order_items          │
│  ├─ id (UUID PK)     │
│  ├─ order_id (FK)    │
│  ├─ product_id (FK)  │
│  ├─ quantity         │
│  └─ price            │
│                      │
│ order_status_hist.   │
│  ├─ id (UUID PK)     │
│  ├─ order_id (FK)    │
│  ├─ status           │
│  └─ timestamp        │
└──────────────────────┘

┌──────────────────────┐
│     users_db         │  Owner: User Service
├──────────────────────┤
│ user_profiles        │
│  ├─ id (UUID PK)     │
│  ├─ user_id (FK)     │
│  ├─ first_name       │
│  ├─ last_name        │
│  └─ phone            │
│                      │
│ addresses            │
│  ├─ id (UUID PK)     │
│  ├─ user_id (FK)     │
│  ├─ street           │
│  ├─ city             │
│  ├─ country          │
│  └─ postal_code      │
│                      │
│ preferences          │
│  ├─ id (UUID PK)     │
│  ├─ user_id (FK)     │
│  ├─ language         │
│  ├─ currency         │
│  └─ notifications    │
└──────────────────────┘
```

---

## 7. Deployment Topology - Kubernetes on AWS EKS

```
┌──────────────────────────────────────────────────────────────────┐
│                     AWS Region: us-east-1                         │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                     AWS VPC: 10.0.0.0/16                          │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  AZ: us-east-1a              AZ: us-east-1b                      │
│  Subnet: 10.0.1.0/24         Subnet: 10.0.2.0/24                │
│  ┌──────────────────┐        ┌──────────────────┐               │
│  │  EKS Worker Node │        │  EKS Worker Node │               │
│  │  (m7i.large)     │        │  (m7i.large)     │               │
│  │                  │        │                  │               │
│  │ Pods:            │        │ Pods:            │               │
│  │ ├─ gateway       │        │ ├─ auth          │               │
│  │ ├─ products      │        │ ├─ orders        │               │
│  │ ├─ users         │        │ └─ prometheus    │               │
│  │ ├─ prometheus    │        │                  │               │
│  │ └─ node-exporter │        │                  │               │
│  └──────────────────┘        └──────────────────┘               │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ RDS PostgreSQL (Multi-AZ)                               │   │
│  │  ├─ Primary (us-east-1a)                                │   │
│  │  └─ Standby (us-east-1b)                                │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ ELB / Load Balancer                                      │   │
│  │  Routes traffic to EKS services                          │   │
│  └─────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│              Service Mesh (Optional - Istio)                      │
├──────────────────────────────────────────────────────────────────┤
│ - Service-to-service TLS                                         │
│ - Traffic management & load balancing                            │
│ - Distributed tracing (Jaeger)                                   │
│ - Security policies                                              │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│           Ingress Controller (NGINX / AWS LB)                    │
├──────────────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────────────┐    │
│ │ TLS Termination (Let's Encrypt certificates)             │    │
│ │ Hostname: boutique.example.com                           │    │
│ └────────────────┬─────────────────────────────────────────┘    │
│                  │                                               │
│        ┌─────────┴──────────┬──────────┐                        │
│        │                    │          │                        │
│        ▼                    ▼          ▼                        │
│   ┌────────┐          ┌────────┐  ┌───────┐                   │
│   │Frontend │          │Gateway │  │Grafana│                   │
│   │Service  │          │Service │  │Service│                   │
│   └────────┘          └────────┘  └───────┘                   │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                   ArgoCD (GitOps)                                │
├──────────────────────────────────────────────────────────────────┤
│ Git Repository (Main Branch)                                     │
│  ├─ k8s/ (Kubernetes manifests)                                 │
│  └─ helm/ (Helm charts)                                         │
│          ▼                                                       │
│ ArgoCD App Controller                                            │
│  - Watches git repo for changes                                 │
│  - Compares desired vs actual state                             │
│  - Automatically syncs changes                                  │
│  - Stores application state in etcd                             │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│              Monitoring & Observability                          │
├──────────────────────────────────────────────────────────────────┤
│ Prometheus                Grafana              CloudWatch        │
│  ├─ ServiceMonitor        ├─ Dashboards       ├─ Logs           │
│  ├─ PrometheusRule        ├─ Alerts           ├─ Metrics        │
│  └─ Retention: 15d        └─ Admin/admin      └─ Traces         │
└──────────────────────────────────────────────────────────────────┘
```

---

## 8. CI/CD Pipeline Flow

```
┌─────────────────────────────────────────────────────────────────┐
│               GitHub Commit to Production                        │
└─────────────────────────────────────────────────────────────────┘

Developer commits to main branch
         ▼
┌──────────────────────────────────────┐
│ GitHub Actions Workflow Triggered    │
│ (.github/workflows/ci.yml)           │
└──────────────┬───────────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
   ┌────────┐      ┌──────────┐
   │ Lint   │      │ Test     │
   │ Code   │      │ Unit     │
   └────┬───┘      └─────┬────┘
        │                │
        └────────┬───────┘
                 ▼
          ┌────────────────┐
          │ Build Docker   │
          │ Images (7x)    │
          └────────┬───────┘
                   ▼
          ┌────────────────────────────────┐
          │ Integration Tests               │
          │ (docker-compose test)           │
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ Push to AWS ECR                │
          │ - Tag with git commit hash    │
          │ - Tag as :latest              │
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ Trigger ArgoCD Sync            │
          │ (via webhook or manual)        │
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ ArgoCD Detects Changes         │
          │ - Pulls latest manifests from git
          │ - Compares with running cluster
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ Deploy to EKS                  │
          │ - Rolling update strategy      │
          │ - 0 downtime deployment        │
          │ - Automatic rollback on failure
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ Health Checks                  │
          │ - Liveness probes              │
          │ - Readiness probes             │
          │ - Service availability checks  │
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ Smoke Tests                    │
          │ - API health endpoints         │
          │ - Database connectivity        │
          │ - External service checks      │
          └────────┬───────────────────────┘
                   ▼
          ┌────────────────────────────────┐
          │ Notification                   │
          │ ✓ Deployment successful        │
          │ ✗ Deployment failed (rollback) │
          └────────────────────────────────┘
```

---

## 9. High Availability & Disaster Recovery

```
┌──────────────────────────────────────────────────────────────────┐
│                 Primary Region (us-east-1)                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Application Services                             │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐               │   │
│  │  │ Service  │  │ Service  │  │ Service  │               │   │
│  │  │ Replica1 │  │ Replica2 │  │ Replica3 │               │   │
│  │  └──────────┘  └──────────┘  └──────────┘               │   │
│  │                                                           │   │
│  │  Auto-Scaling:                                           │   │
│  │  Min: 2, Desired: 3, Max: 10                             │   │
│  │                                                           │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Database (Primary) - Multi-AZ                            │   │
│  │  ┌────────────────┐         ┌────────────────┐           │   │
│  │  │ Master Instance│   Sync  │ Standby        │           │   │
│  │  │ us-east-1a    ├─────────→│ us-east-1b    │           │   │
│  │  └────────────────┘         └────────────────┘           │   │
│  │                                                           │   │
│  │  Backups:                                                │   │
│  │  - Hourly snapshots (30 day retention)                   │   │
│  │  - Transaction logs (PITR: 30 days)                      │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
                          │
                          │ Cross-region
                          │ replication
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│              Secondary Region (us-west-2) - Standby             │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Replica Services (Cold Standby)                          │   │
│  │  ├─ EKS Cluster (scaled down)                            │   │
│  │  ├─ Read replicas of database                            │   │
│  │  └─ Infrastructure ready                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  Failover: Manual (< 5 minutes)                                 │
│  - Route53 DNS update                                           │
│  - Scale up replica services                                    │
│  - Promote read replicas                                        │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘

RTO (Recovery Time Objective): 15 minutes
RPO (Recovery Point Objective):  5 minutes
```

---

## 10. Performance Metrics & KPIs

```
┌────────────────────────────────────────────────────────────────┐
│              Boutique System Performance Targets                │
└────────────────────────────────────────────────────────────────┘

API Performance:
┌─────────────────────────┬──────────┬───────────────────┐
│ Metric                  │ Target   │ Measurement Tool  │
├─────────────────────────┼──────────┼───────────────────┤
│ P50 Latency             │ < 50ms   │ Prometheus        │
│ P95 Latency             │ < 200ms  │ Prometheus        │
│ P99 Latency             │ < 500ms  │ Prometheus        │
│ Error Rate              │ < 0.1%   │ Prometheus        │
│ RPS (Requests/sec)      │ 1000+    │ Prometheus        │
└─────────────────────────┴──────────┴───────────────────┘

Database Performance:
┌─────────────────────────┬──────────┬───────────────────┐
│ Metric                  │ Target   │ Measurement Tool  │
├─────────────────────────┼──────────┼───────────────────┤
│ Query Latency (avg)     │ < 10ms   │ PostgreSQL logs   │
│ Connection Pool Usage   │ < 80%    │ Prometheus        │
│ Cache Hit Ratio         │ > 95%    │ Redis metrics     │
│ Replication Lag         │ < 100ms  │ pgAdmin/Prometheus│
└─────────────────────────┴──────────┴───────────────────┘

Availability & Reliability:
┌─────────────────────────┬──────────┬───────────────────┐
│ Metric                  │ Target   │ Measurement Tool  │
├─────────────────────────┼──────────┼───────────────────┤
│ Uptime SLO              │ 99.9%    │ CloudWatch        │
│ MTTR (Mean Time to Rec) │ 15 mins  │ Incident tracking │
│ Deployment Frequency    │ 10/day   │ ArgoCD / GitHub   │
│ Lead Time (Deploy)      │ 15 mins  │ GitHub Actions    │
│ Change Failure Rate     │ < 5%     │ Manual tracking   │
└─────────────────────────┴──────────┴───────────────────┘

Resource Utilization:
┌─────────────────────────┬──────────┬───────────────────┐
│ Metric                  │ Target   │ Measurement Tool  │
├─────────────────────────┼──────────┼───────────────────┤
│ CPU Utilization         │ 40-70%   │ Prometheus        │
│ Memory Utilization      │ 50-75%   │ Prometheus        │
│ Disk Usage              │ < 80%    │ Prometheus        │
│ Network I/O             │ < 50%    │ CloudWatch        │
└─────────────────────────┴──────────┴───────────────────┘

Prometheus Queries for Monitoring:
┌────────────────────────────────────────────────────────┐
│ P95 Latency (request_duration_seconds)                │
│ histogram_quantile(0.95,                              │
│   rate(http_request_duration_seconds_bucket[5m]))    │
│                                                        │
│ Request Rate (RPS)                                    │
│ rate(http_requests_total[1m])                         │
│                                                        │
│ Error Rate                                            │
│ rate(http_requests_total{status=~"5.."}[5m]) /       │
│ rate(http_requests_total[5m])                         │
│                                                        │
│ Service Availability                                  │
│ up{job="gateway"}                                      │
│ (1 = up, 0 = down)                                    │
└────────────────────────────────────────────────────────┘
```

---

## Summary

The Boutique Microservices architecture demonstrates:

1. **Scalability** - Horizontal auto-scaling via Kubernetes
2. **Reliability** - Multi-AZ deployment with failover
3. **Observability** - Comprehensive monitoring and tracing
4. **Automation** - GitOps-driven continuous deployment
5. **Security** - Defense-in-depth with TLS, JWT, RBAC
6. **Maintainability** - Clear separation of concerns, documented APIs
7. **Cost Efficiency** - Right-sized resources with autoscaling

This architecture is production-ready and suitable for enterprise applications!
