# Quick Start Guide - Boutique Microservices

**For busy DevOps professionals - get running in 5 minutes!**

## TL;DR - One Command Start

```bash
cd projects/boutique-microservices
docker-compose up -d --build
```

Done! Your application is running. Access it below.

---

## What Just Happened?

You started 9 containerized services:
- ✅ Frontend (React)
- ✅ API Gateway + 5 backend microservices
- ✅ PostgreSQL database (5 databases)
- ✅ Prometheus monitoring
- ✅ Grafana dashboards

All fully networked and ready to use.

---

## Access Your Application

| Component | URL | Username/Password |
|-----------|-----|-------------------|
| **Frontend** | http://localhost:3000 | (No login required) |
| **API Gateway** | http://localhost:3001 | API endpoint |
| **Prometheus** | http://localhost:9090 | (No auth) |
| **Grafana** | http://localhost:3007 | admin / admin |

---

## Test the Application

### 1. Test Frontend
```bash
# Open in browser
http://localhost:3000
```

### 2. Test API Health
```bash
curl http://localhost:3001/health
# Expected: {"status":"ok","service":"gateway"}
```

### 3. Test Auth Service
```bash
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'
```

### 4. View Metrics
```bash
curl http://localhost:3001/metrics | head -20
```

### 5. Check Prometheus
```bash
# Open in browser
http://localhost:9090/targets
# Should see all 6 services as "UP" (green)
```

---

## Useful Commands

### Monitor Services
```bash
# Watch all containers
docker-compose ps

# Follow logs in real-time
docker-compose logs -f

# Logs for specific service
docker-compose logs -f gateway

# View last 50 lines
docker-compose logs --tail 50 auth
```

### Database Operations
```bash
# Connect to PostgreSQL
docker-compose exec postgres psql -U postgres -d auth_db

# List databases
docker-compose exec postgres psql -U postgres -l

# Run SQL query
docker-compose exec postgres psql -U postgres -d products_db \
  -c "SELECT COUNT(*) FROM products;"
```

### Docker Management
```bash
# Stop all services (keep data)
docker-compose down

# Stop and remove everything (reset)
docker-compose down -v

# Rebuild specific service
docker-compose up -d --build gateway

# See resource usage
docker stats
```

### Troubleshooting
```bash
# Check service connectivity
docker-compose exec gateway curl http://auth:3002/health

# View database logs
docker-compose logs postgres | grep ERROR

# Restart a service
docker-compose restart auth

# View Docker disk usage
docker system df

# Clean unused images/volumes
docker system prune -a
```

---

## Common Scenarios

### Scenario 1: "Frontend not loading"

**Check:**
```bash
# Is frontend running?
docker-compose ps frontend

# Check frontend logs
docker-compose logs frontend

# Is port 3000 available?
lsof -i :3000

# Test API connection
curl http://localhost:3001/health
```

**Fix:**
```bash
docker-compose restart frontend
```

### Scenario 2: "API calls returning 502"

**Check:**
```bash
# Is gateway running?
docker-compose ps gateway

# Can gateway reach auth service?
docker-compose exec gateway curl http://auth:3002/health

# Check gateway logs
docker-compose logs gateway | grep error
```

**Fix:**
```bash
# Restart gateway and dependencies
docker-compose restart gateway auth product-service
```

### Scenario 3: "Database connection failed"

**Check:**
```bash
# Is database healthy?
docker-compose ps postgres

# Is it ready?
docker-compose exec postgres pg_isready -U postgres

# Check database logs
docker-compose logs postgres
```

**Fix:**
```bash
# Restart database
docker-compose restart postgres

# Wait for services to reconnect
sleep 10
docker-compose logs postgres | tail -20
```

### Scenario 4: "Out of memory"

**Check:**
```bash
# See memory usage
docker stats

# Check Docker system
docker system df
```

**Fix:**
```bash
# Increase Docker memory (depends on OS)
# Or restart and reduce number of services
docker-compose down
docker-compose up -d --scale frontend=1
```

---

## Monitoring Dashboard

### Pre-configured Grafana Dashboards

1. **Service Metrics Overview**
   - Request rate by service
   - Error rate tracking
   - Latency distribution

2. **API Gateway Performance**
   - Requests per endpoint
   - Response times
   - Status code distribution

3. **Service Health Status**
   - Uptime percentage
   - Active connections
   - Memory/CPU usage

### Custom Metrics Queries

**Grafana → Explore → Prometheus**

```promql
# Requests per second by service
rate(http_requests_total[5m])

# P95 latency by service
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# Memory usage
process_resident_memory_bytes / 1024 / 1024

# Node.js event loop lag
nodejs_eventloop_lag_seconds
```

---

## Production Checklist

Before deploying to production:

- [ ] Change default database password
- [ ] Update JWT secret key
- [ ] Enable HTTPS/TLS
- [ ] Set up proper logging aggregation (ELK, etc.)
- [ ] Configure database backups
- [ ] Set up alerting for critical metrics
- [ ] Test disaster recovery procedure
- [ ] Document runbooks for common issues
- [ ] Set up rate limiting on API
- [ ] Enable authentication on Grafana/Prometheus
- [ ] Configure CORS properly for your domain
- [ ] Set up auto-scaling policies

---

## Next Steps

1. **Explore the code:**
   - Frontend: `frontend/src/`
   - Backend: `backend/services/*/src/`
   - Infrastructure: `projects/Infrastructure/`

2. **Read full documentation:**
   - See `PROJECT_DOCUMENTATION.md`

3. **Deploy to cloud:**
   - See `projects/Infrastructure/README.md`

4. **Set up CI/CD:**
   - Configure GitHub Actions
   - Set up ArgoCD

5. **Customize for your needs:**
   - Add your own services
   - Modify database schemas
   - Add new routes

---

## Support & Debugging

### Check System Requirements

```bash
# Docker version (should be 19.03+)
docker --version

# Docker Compose version (should be 1.27+)
docker-compose --version

# Available disk space (need 5GB+)
df -h

# Available memory (need 4GB+)
free -h
```

### Full Diagnostics

```bash
#!/bin/bash
echo "=== System Info ==="
docker --version
docker-compose --version

echo "=== Running Containers ==="
docker-compose ps

echo "=== Service Health ==="
curl -s http://localhost:3001/health | jq .
curl -s http://localhost:9090/-/healthy | head -5

echo "=== Database Status ==="
docker-compose exec postgres pg_isready -U postgres

echo "=== Resource Usage ==="
docker stats --no-stream

echo "=== Error Log Summary ==="
docker-compose logs | grep -i error | tail -5
```

### Performance Baseline

```bash
# Measure API response time
time curl http://localhost:3001/health

# Load test (requires ab or vegeta)
ab -n 1000 -c 10 http://localhost:3001/health

# Monitor during load test
watch docker stats
```

---

## Key Ports

| Port | Service | URL |
|------|---------|-----|
| 3000 | Frontend | http://localhost:3000 |
| 3001 | Gateway | http://localhost:3001 |
| 3002 | Auth | http://localhost:3002 |
| 3003 | Products | http://localhost:3003 |
| 3004 | Orders | http://localhost:3004 |
| 3005 | Order History | http://localhost:3005 |
| 3006 | Users | http://localhost:3006 |
| 5432 | PostgreSQL | localhost:5432 |
| 9090 | Prometheus | http://localhost:9090 |
| 3007 | Grafana | http://localhost:3007 |

---

## Still Need Help?

1. Check logs: `docker-compose logs -f`
2. See full documentation: `PROJECT_DOCUMENTATION.md`
3. Review GitHub issues: (if public repo)
4. Ask your team DevOps lead

**Remember:** This is a learning project showing best practices. For production, add:
- Authentication on monitoring dashboards
- SSL/TLS certificates
- Database backups
- Disaster recovery procedures
- Incident response runbooks

---

Happy containerizing! 🐳
