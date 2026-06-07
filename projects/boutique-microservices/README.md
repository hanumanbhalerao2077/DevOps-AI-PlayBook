# Boutique Microservices - Enterprise DevOps Project

<img src="https://img.shields.io/badge/status-production%20ready-brightgreen" alt="Status">
<img src="https://img.shields.io/badge/docker-v20%2B-blue" alt="Docker">
<img src="https://img.shields.io/badge/kubernetes-v1.28%2B-blue" alt="Kubernetes">
<img src="https://img.shields.io/badge/license-MIT-green" alt="License">

**A comprehensive, production-grade microservices e-commerce platform demonstrating modern DevOps best practices.**

---

## 🎯 Project Overview

**Boutique** is a luxury e-commerce microservices application built with enterprise-grade DevOps practices. It showcases a complete stack from local containerization to cloud-native deployment on AWS EKS with GitOps automation.

### What You Get

✅ **6 Microservices** - Authentication, Product Catalog, Shopping Cart, Orders, User Management, API Gateway  
✅ **React Frontend** - Modern UI with Material-UI components  
✅ **PostgreSQL** - 5 specialized databases with proper data models  
✅ **Container Orchestration** - Docker Compose (local) & Kubernetes (cloud)  
✅ **GitOps Pipeline** - ArgoCD for continuous, declarative deployment  
✅ **Infrastructure as Code** - Terraform modules for AWS EKS  
✅ **Monitoring & Observability** - Prometheus + Grafana  
✅ **Complete Documentation** - Architecture guides, setup instructions, interview talking points  

---

## 📊 Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│                    End Users (Web)                        │
└────────────────────────────┬─────────────────────────────┘
                             │
                    HTTP/REST │ Calls
                             ▼
                  ┌──────────────────────┐
                  │  React Frontend      │
                  │  (Port 3000)         │
                  └──────────┬───────────┘
                             │
                  ┌──────────▼───────────┐
                  │   API Gateway        │
                  │   (Port 3001)        │
                  │  - Routing           │
                  │  - Security          │
                  │  - Monitoring        │
                  └──────────┬───────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼──┐           ┌───▼───┐           ┌──▼─────┐
    │ Auth  │           │Product│           │ Orders │
    │3002   │           │3003   │           │ 3005   │
    └────┬──┘           └───┬───┘           └────┬───┘
         │                  │                    │
         └──────────┬───────┴────────────────────┘
                    │
            ┌───────▼────────┐
            │ PostgreSQL     │
            │ (5 databases) │
            └────────────────┘

    ┌─────────────────────────────────┐
    │   Monitoring                    │
    │ Prometheus ↔ Grafana            │
    └─────────────────────────────────┘
```

**For detailed architecture:**
- See [ARCHITECTURE.md](./ARCHITECTURE.md) - Complete system design
- Service interaction diagrams
- Data flow visualizations
- Kubernetes deployment topology
- CI/CD pipeline flow

---

## 🚀 Quick Start

### Prerequisites

- **Docker** 19.03+
- **Docker Compose** 1.27+
- **Node.js** 20 LTS+
- **Git**

### Start in 30 Seconds

```bash
# Clone and navigate
git clone <repo-url>
cd projects/boutique-microservices

# Start all services
docker-compose up -d --build

# Wait for services to initialize (30-45 seconds)
docker-compose ps
```

### Access the Application

| Service | URL | Use |
|---------|-----|-----|
| **Frontend** | http://localhost:3000 | Customer web app |
| **API Gateway** | http://localhost:3001 | API endpoint |
| **Prometheus** | http://localhost:9090 | Metrics browser |
| **Grafana** | http://localhost:3007 | Dashboards (admin/admin) |

**For complete setup guide:**
- See [QUICK_START.md](./QUICK_START.md) - 5-minute setup
- Troubleshooting guide
- Common scenarios

---

## 📚 Comprehensive Documentation

### Core Documentation

1. **[PROJECT_DOCUMENTATION.md](./PROJECT_DOCUMENTATION.md)** (45+ pages)
   - Complete project overview
   - All components & services detailed
   - Technology stack breakdown
   - Setup & deployment instructions
   - Monitoring strategies
   - CI/CD pipeline explanation
   - Production checklist
   - **Resume bullet points for interviews**

2. **[ARCHITECTURE.md](./ARCHITECTURE.md)**
   - 10 detailed architecture diagrams
   - System design with ASCII art
   - Data flow visualizations
   - Service communication patterns
   - Database schema overview
   - Kubernetes deployment topology
   - Performance metrics

3. **[QUICK_START.md](./QUICK_START.md)**
   - 5-minute startup guide
   - Common scenarios & fixes
   - Monitoring dashboard access
   - Production checklist

### Infrastructure Documentation

- **[projects/Infrastructure/README.md](../Infrastructure/README.md)**
  - Terraform deployment guide
  - AWS resources provisioned
  - EKS cluster setup
  - Cost estimation

- **[gitops/README.md](../gitops/README.md)**
  - ArgoCD configuration
  - Kubernetes manifests
  - GitOps workflow
  - Continuous deployment

---

## 🛠️ Project Structure

```
boutique-microservices/
├── backend/
│   ├── services/
│   │   ├── gateway/          # API Gateway (Express.js)
│   │   ├── auth/             # Authentication Service
│   │   ├── product-service/  # Product Catalog
│   │   ├── order-service/    # Shopping Cart
│   │   ├── orders/           # Order History
│   │   └── user-service/     # User Management
│   ├── shared/               # Shared TypeScript types
│   └── package.json          # npm workspace config
│
├── frontend/                 # React 19.2 application
│   ├── src/
│   │   ├── components/       # React components
│   │   ├── pages/            # Page components
│   │   ├── services/         # API client
│   │   └── contexts/         # State management
│   ├── public/               # Static assets
│   └── package.json
│
├── database/
│   ├── init/                 # Database initialization
│   │   ├── 10-create-databases.sh
│   │   └── 20-init-schema.sql
│   ├── boutique_full.sql     # Sample data
│   └── quick-seed.sql
│
├── docker-compose.yml        # Local orchestration
├── prometheus/               # Monitoring config
├── grafana/                  # Dashboard provisioning
│
└── docs/
    ├── PROJECT_DOCUMENTATION.md  # 45+ page guide
    ├── ARCHITECTURE.md            # System design
    ├── QUICK_START.md             # 5-min setup
    └── README.md                  # This file

infrastructure/              # See ../Infrastructure/
gitops/                      # See ../gitops/
```

---

## 🎯 Key Features

### Architecture Features

✅ **Microservices Design**
- 6 independent services with clear boundaries
- Database per service pattern
- Asynchronous communication ready

✅ **Container Native**
- Multi-stage Docker builds (70% size reduction)
- Health checks on all services
- Non-root user execution
- Efficient layer caching

✅ **Kubernetes Ready**
- Stateless service design
- ConfigMaps & Secrets support
- Horizontal pod autoscaling
- Rolling updates

✅ **Production Observability**
- Prometheus metrics export
- Grafana dashboards
- Request tracing
- Performance monitoring

### Developer Features

✅ **Local Development**
- Docker Compose setup in one command
- Hot-reload with nodemon
- Concurrent service execution
- Database migrations included

✅ **CI/CD Ready**
- GitHub Actions workflow
- Automated testing
- Docker image building
- ECR push pipeline

✅ **Infrastructure as Code**
- Terraform modules
- Kubernetes manifests
- Kustomize overlays
- Automated infrastructure provisioning

---

## 💾 Technology Stack

### Backend
- **Language:** TypeScript 5.2.2
- **Runtime:** Node.js 20 LTS
- **Framework:** Express.js 4.18.2
- **Database:** PostgreSQL 15-alpine
- **Auth:** JWT + bcryptjs
- **Container:** Docker + Docker Compose

### Frontend
- **Framework:** React 19.2
- **UI Library:** Material-UI 7.3
- **Build Tool:** Create React App 5.0.1
- **HTTP Client:** Axios 1.13.4
- **Routing:** React Router 7.13.0

### Infrastructure & DevOps
- **Container Orchestration:** Docker Compose (local), Kubernetes (cloud)
- **IaC:** Terraform 1.0+
- **GitOps:** ArgoCD 2.0+
- **Monitoring:** Prometheus, Grafana
- **Cloud:** AWS (EKS, ECR, RDS, S3)
- **CI/CD:** GitHub Actions

### Database
- **Primary:** PostgreSQL 15
- **Pattern:** Database per service (5 databases)
- **Backup:** Automated snapshots
- **Replication:** Multi-AZ ready

---

## 📖 Setup Instructions

### Option 1: Docker Compose (Recommended for Interviews)

**Fastest way to get running:**

```bash
cd projects/boutique-microservices
docker-compose up -d --build
```

**Verify all services:**
```bash
docker-compose ps
```

**View logs:**
```bash
docker-compose logs -f
```

**Stop services:**
```bash
docker-compose down
```

### Option 2: Local Node.js Development

**For development with auto-reload:**

```bash
# Install dependencies
npm run install:all

# Start PostgreSQL in Docker
docker run -d --name boutique-postgres \
  -e POSTGRES_PASSWORD=postgres123 \
  -p 5432:5432 \
  postgres:15-alpine

# Start all services with nodemon
npm run dev
```

### Option 3: AWS EKS Deployment

**For production cloud deployment:**

```bash
cd projects/Infrastructure

# Initialize Terraform
terraform init

# Plan deployment
terraform plan

# Apply changes (creates VPC, EKS, ECR, ArgoCD)
terraform apply

# Deploy services via ArgoCD
cd ../gitops
argocd app sync boutique
```

**For complete deployment guide:**
- See [projects/Infrastructure/README.md](../Infrastructure/README.md)

---

## 🧪 Testing the Application

### API Health Check

```bash
curl http://localhost:3001/health
# Expected: {"status":"ok","service":"gateway"}
```

### Frontend Access

```bash
# Open in browser
http://localhost:3000
```

### Metrics & Monitoring

```bash
# Prometheus (raw metrics)
curl http://localhost:3001/metrics

# Prometheus UI (query metrics)
http://localhost:9090

# Grafana (visualized metrics)
http://localhost:3007
# Login: admin / admin
```

### Database Operations

```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U postgres -d auth_db

# List all databases
docker-compose exec postgres psql -U postgres -l

# Run SQL query
docker-compose exec postgres psql -U postgres -d products_db \
  -c "SELECT COUNT(*) FROM products;"
```

---

## 🔍 Common Commands

### Docker Compose Management

```bash
# Start all services
docker-compose up -d

# Rebuild images
docker-compose up -d --build

# View status
docker-compose ps

# Follow logs
docker-compose logs -f

# Logs for specific service
docker-compose logs -f gateway

# Stop all services
docker-compose down

# Reset everything (remove volumes)
docker-compose down -v

# Restart specific service
docker-compose restart auth
```

### Development

```bash
# Install all dependencies
npm run install:all

# Build all services
npm run build

# Run tests
npm run test

# Lint code
npm run lint

# Type check
npm run type-check
```

### Database

```bash
# Initialize databases
docker-compose exec postgres bash /docker-entrypoint-initdb.d/10-create-databases.sh

# Backup database
docker-compose exec postgres pg_dump -U postgres auth_db > backup.sql

# Restore database
docker-compose exec -T postgres psql -U postgres auth_db < backup.sql
```

### Monitoring

```bash
# Resource usage
docker stats

# View system info
docker system df

# Prune unused containers
docker system prune -a
```

---

## 🚨 Troubleshooting

### Issue: Port already in use

```bash
# Find process using port
lsof -i :3000

# Kill the process
kill -9 <PID>

# Or change port mapping in docker-compose.yml
```

### Issue: Database connection refused

```bash
# Check if postgres is healthy
docker-compose ps postgres

# Wait for postgres to be ready
docker-compose exec postgres pg_isready -U postgres

# Restart postgres
docker-compose restart postgres
```

### Issue: Services can't communicate

```bash
# Test connectivity from gateway
docker-compose exec gateway curl http://auth:3002/health

# Check logs for errors
docker-compose logs --tail 50 gateway

# Verify service names match
# All services must use container service names (not localhost)
```

**For more troubleshooting:**
- See [QUICK_START.md](./QUICK_START.md#troubleshooting)
- See [PROJECT_DOCUMENTATION.md](./PROJECT_DOCUMENTATION.md#known-issues--solutions)

---

## 📈 Monitoring & Observability

### Pre-configured Dashboards

**Grafana (http://localhost:3007)**
- Service metrics overview
- API gateway performance
- Database performance
- Resource utilization

### Custom Prometheus Queries

```promql
# Request rate (RPS)
rate(http_requests_total[1m])

# P95 latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# Memory usage
process_resident_memory_bytes / 1024 / 1024
```

### Viewing Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f gateway

# Follow with timestamps
docker-compose logs -f --timestamps

# Last 100 lines
docker-compose logs --tail 100 auth
```

---

## 🔐 Security Best Practices

✅ **Authentication & Authorization**
- JWT tokens with HS256 signing
- bcryptjs password hashing
- Session management

✅ **Data Security**
- Database credentials from environment variables
- Never commit secrets to Git
- Use AWS Secrets Manager in production

✅ **API Security**
- Helmet.js for security headers
- CORS properly configured
- Input validation with Joi
- SQL injection prevention with parameterized queries

✅ **Container Security**
- Non-root user execution
- Multi-stage builds
- Minimal base images (alpine)
- Regular dependency updates

✅ **Network Security**
- HTTPS/TLS in production
- Service-to-service encryption (optional with Istio)
- Network policies on Kubernetes

---

## 🎓 Learning Outcomes

After working with this project, you'll understand:

### Architecture & Design
- ✅ Microservices architecture patterns
- ✅ API Gateway pattern
- ✅ Database per service pattern
- ✅ Service communication (REST, async)

### DevOps & Infrastructure
- ✅ Docker containerization & optimization
- ✅ Container orchestration (Compose, Kubernetes)
- ✅ Infrastructure as Code (Terraform)
- ✅ GitOps workflows (ArgoCD)

### Cloud & AWS
- ✅ AWS EKS (Elastic Kubernetes Service)
- ✅ ECR (Elastic Container Registry)
- ✅ RDS (Relational Database Service)
- ✅ VPC networking & security groups

### Observability & Monitoring
- ✅ Prometheus metrics collection
- ✅ Grafana dashboard creation
- ✅ Application instrumentation
- ✅ Performance monitoring

### CI/CD & Automation
- ✅ GitHub Actions workflows
- ✅ Automated testing
- ✅ Docker image building
- ✅ Continuous deployment

---

## 💼 Interview Preparation

This project is **interview-ready** and demonstrates enterprise DevOps practices.

### Talking Points

**Architecture:**
- "I designed a microservices architecture with 6 independent services..."
- "Each service owns its database following the database-per-service pattern..."
- "The API Gateway routes requests and handles cross-cutting concerns..."

**Deployment:**
- "We use Docker Compose for local development and Kubernetes for production..."
- "Infrastructure is defined as code using Terraform..."
- "We follow GitOps practices with ArgoCD for continuous deployment..."

**Scaling:**
- "Services scale horizontally on Kubernetes with auto-scaling policies..."
- "Database uses read replicas and connection pooling..."
- "We cache frequently accessed data to reduce database load..."

**Monitoring:**
- "Prometheus scrapes metrics every 15 seconds..."
- "Grafana provides real-time visibility into system health..."
- "We alert on SLOs including 99.9% uptime and p95 latency..."

**See [PROJECT_DOCUMENTATION.md](./PROJECT_DOCUMENTATION.md#resume-bullet-points) for:**
- 20+ professional resume bullet points
- Detailed technical explanations
- Interview question answers
- Common follow-up questions

---

## 📝 Documentation Files

| File | Purpose | Audience |
|------|---------|----------|
| [PROJECT_DOCUMENTATION.md](./PROJECT_DOCUMENTATION.md) | Complete guide (45+ pages) | Technical deep dive |
| [ARCHITECTURE.md](./ARCHITECTURE.md) | System design with diagrams | Visual learners |
| [QUICK_START.md](./QUICK_START.md) | 5-minute setup guide | Quick start |
| [README.md](./README.md) | This file | Overview |
| [../Infrastructure/README.md](../Infrastructure/README.md) | Terraform deployment | Cloud setup |
| [../gitops/README.md](../gitops/README.md) | ArgoCD & Kubernetes | K8s deployment |

---

## 🤝 Contributing

This is a learning/portfolio project. Feel free to:
- Fork and customize
- Add new services
- Implement additional features
- Deploy to your own AWS account
- Extend monitoring & observability

### Suggestions for Enhancement
- Add message queue (RabbitMQ/SQS)
- Implement service mesh (Istio)
- Add distributed tracing (Jaeger)
- Implement API rate limiting
- Add load testing (k6/Apache Bench)
- Implement blue-green deployments
- Add API documentation (OpenAPI/Swagger)
- Add end-to-end tests

---

## 📊 Project Stats

| Metric | Value |
|--------|-------|
| Backend Services | 6 |
| Frontend Framework | React 19.2 |
| Container Services | 9 |
| Kubernetes Manifests | 12+ |
| Database Schemas | 5 |
| Terraform Modules | 4 |
| Lines of Code | 5,000+ |
| Documentation | 15,000+ lines |
| Setup Time | < 5 minutes |
| Production Readiness | ✅ Yes |

---

## 🏆 What Makes This Project Special

1. **Complete Stack** - From code to cloud to monitoring
2. **Production Ready** - Not a toy project; real enterprise patterns
3. **Well Documented** - 45+ pages of documentation
4. **Interview Ready** - Resume points, talking points, architecture explanations
5. **Hands-on Learning** - Understand each layer thoroughly
6. **Scalable** - Grows from your laptop to AWS EKS
7. **Modern Tech** - Latest versions of React, Node, Kubernetes, Terraform

---

## 📞 Support & Resources

### Getting Help
1. Check [QUICK_START.md](./QUICK_START.md) for common issues
2. Review [PROJECT_DOCUMENTATION.md](./PROJECT_DOCUMENTATION.md) for detailed explanations
3. See [ARCHITECTURE.md](./ARCHITECTURE.md) for system design understanding

### Useful Links
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

---

## 📄 License

MIT License - Free to use, modify, and distribute.

---

## 🎉 Ready to Get Started?

### Quick Commands to Get Running

```bash
# 1. Navigate to project
cd projects/boutique-microservices

# 2. Start all services
docker-compose up -d --build

# 3. Wait for initialization (30-45 seconds)
sleep 45

# 4. Verify services are running
docker-compose ps

# 5. Access the application
echo "Frontend: http://localhost:3000"
echo "Grafana: http://localhost:3007 (admin/admin)"
echo "Prometheus: http://localhost:9090"
```

**Next Steps:**
1. Read [QUICK_START.md](./QUICK_START.md) (5 minutes)
2. Explore [PROJECT_DOCUMENTATION.md](./PROJECT_DOCUMENTATION.md) (30 minutes)
3. Review [ARCHITECTURE.md](./ARCHITECTURE.md) (30 minutes)
4. Deploy to AWS EKS (1-2 hours)

---

**Built with ❤️ for DevOps Engineers**

*Last Updated: June 2026*  
*Version: 1.0*  
*Status: Production Ready* ✅
