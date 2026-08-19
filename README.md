# site

Repo contains infrastructure, Keycloak and databases used by the rest of the site services.

## Docker Compose (existing workflow)

1. Copy environment file:

       copy .env.dist .env

2. Start the production-like stack:

       docker compose up -d

3. Start the dev stack:

       docker compose -f docker-compose-dev.yml up -d

After Keycloak starts, open `http://localhost:8080`, add/import realm `dev-realm-export.json`, then create a user in the new realm with at least `USER` role.

## Kubernetes production deployment

Production manifests are in `k8s\production` and include:

- nginx proxy/static site
- Keycloak
- PostgreSQL for backend
- PostgreSQL for Keycloak
- Redis cache
- Kafka queue
- Prometheus ServiceMonitors + health alerts
- Ingress + PodDisruptionBudgets + probes + persistent volumes

### 1. Configure secrets and TLS

1. Create namespace first:

       kubectl apply -f k8s\production\namespace.yaml

2. Create secret manifest from template and replace values:

       copy k8s\production\secrets-template.yaml k8s\production\secrets.yaml

3. Apply secret:

       kubectl apply -f k8s\production\secrets.yaml

4. Create TLS secret used by ingress:

       kubectl -n site create secret tls site-tls --cert=fullchain.pem --key=privkey.pem

### 2. Deploy

Apply everything:

    kubectl apply -k k8s\production

### 3. Run / smoke test

1. Check pods:

       kubectl -n site get pods

2. Check ingress and services:

       kubectl -n site get ingress,svc

3. Basic health check through port-forward:

       kubectl -n site port-forward svc/nginx 8080:80
       curl http://localhost:8080/nginx-health

4. Keycloak internal health:

       kubectl -n site port-forward svc/keycloak 8081:8080
       curl http://localhost:8081/health/ready

### 4. Queue and cache endpoints

- Redis: `redis.site.svc.cluster.local:6379`
- Kafka bootstrap servers: `kafka.site.svc.cluster.local:9092`

### 5. Prometheus health monitoring

`k8s\production\prometheus-monitoring.yaml` adds:

- ServiceMonitor for Keycloak metrics (`/metrics`)
- ServiceMonitor for Kafka metrics (through `kafka-exporter`)
- PrometheusRule alerts if either target is down for 5 minutes

This requires Prometheus Operator CRDs (for example `kube-prometheus-stack`).

### 6. Frontend assets

The nginx deployment expects site frontend files in PVC `site-frontend-assets` mounted at `/usr/share/site-frontend`.
Publish your frontend build output into that volume as part of your CI/CD pipeline.

### 7. GitHub Actions — automated deployment

The workflow `.github/workflows/deploy-k8s.yml` runs on every push to `main` and applies the full k8s stack automatically.

Configure the following secrets in **GitHub → Settings → Secrets and variables → Actions**:

| Secret | Description |
|--------|-------------|
| `KUBECONFIG_B64` | Base64-encoded kubeconfig: `base64 -w0 ~/.kube/config` |
| `DB_PASS` | PostgreSQL password |
| `KC_PASS` | Keycloak admin password |
| `POSTGRES_USER` | PostgreSQL username |
| `REDIS_PASSWORD` | Redis password |
| `KAFKA_BOOTSTRAP_SERVERS` | Kafka connection string, e.g. `kafka.site.svc.cluster.local:9092` |

### 8. Deploy updates manually

When you change manifests:

    kubectl apply -k k8s\production

When you update images used by this stack, update image tags in manifests and re-apply.
