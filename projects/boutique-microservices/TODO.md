# Boutique Microservices - Docker Compose Local Run TODO

## Planned fixes
- [ ] Update Dockerfiles to avoid failing `npm install` during image build (use runtime install/start or prebuilt JS entrypoints).
- [ ] Update `docker-compose.yml` to remove/limit build usage and ensure correct ports/env wiring.
- [ ] Ensure DB schema/init runs before services (postgres init + `depends_on` health).
- [ ] Ensure gateway proxies correctly to auth/products/orders/users with correct service URLs.
- [ ] Ensure each service exposes `/health`.
- [ ] Frontend is served reliably (prefer existing `frontend/build` or build at runtime / switch to `npm start` container dev).

## Validation
- [ ] `docker compose up -d` completes without errors.
- [ ] All containers are healthy / running.
- [ ] Run curl health checks for gateway/auth/products/orders/orders-service/user-service.
- [ ] Verify end-to-end: `curl http://localhost:3001/api/products`.
- [ ] Verify frontend loads: `curl http://localhost:3000/` and `/products`.

