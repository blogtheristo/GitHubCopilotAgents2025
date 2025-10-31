# Deployment Guide

## Overview
This guide walks through deploying an AI agent with private infrastructure, ensuring all processing remains under your control.

## Prerequisites

### Infrastructure Requirements
- Private cloud (AWS, Azure, GCP) or on-premises servers
- Kubernetes cluster (recommended) or Docker
- Database (PostgreSQL, MongoDB)
- Message queue (RabbitMQ, Kafka)
- Object storage (S3, Azure Blob, MinIO)

### Software Requirements
- Python 3.9+
- Docker & Docker Compose
- Kubernetes (kubectl) - optional
- Git

## Deployment Options

### Option 1: Docker Compose (Quick Start)

```bash
# 1. Clone repository
git clone https://github.com/yourusername/agent-deployment
cd agent-deployment

# 2. Configure environment
cp .env.example .env
nano .env  # Edit with your settings

# 3. Deploy with Docker Compose
docker-compose up -d

# 4. Verify deployment
docker-compose ps
docker-compose logs -f agent
```

**docker-compose.yaml**
```yaml
version: '3.8'

services:
  agent:
    build: ./agent
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - AGENT_CONFIG_PATH=/config/agent-config.yaml
    volumes:
      - ./config:/config
      - ./data:/data
    ports:
      - "8080:8080"
    depends_on:
      - database
      - redis
  
  database:
    image: postgres:15
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=agent_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
  
  monitoring:
    image: prom/prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
```

### Option 2: Kubernetes (Production)

```bash
# 1. Create namespace
kubectl create namespace agent-system

# 2. Deploy configuration
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml

# 3. Deploy agent
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# 4. Verify deployment
kubectl get pods -n agent-system
kubectl logs -f deployment/agent -n agent-system
```

**k8s/deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent
  namespace: agent-system
spec:
  replicas: 3
  selector:
    matchLabels:
      app: agent
  template:
    metadata:
      labels:
        app: agent
    spec:
      containers:
      - name: agent
        image: your-registry/agent:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: agent-secrets
              key: database-url
        - name: AGENT_CONFIG
          valueFrom:
            configMapKeyRef:
              name: agent-config
              key: config.yaml
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: agent-service
  namespace: agent-system
spec:
  selector:
    app: agent
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

**k8s/hpa.yaml** (Auto-scaling)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: agent-hpa
  namespace: agent-system
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: agent
  minReplicas: 2
  maxReplicas: 10
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
```

## Configuration

### Environment Variables
```bash
# .env file
DATABASE_URL=postgresql://user:pass@localhost:5432/agent_db
REDIS_URL=redis://localhost:6379/0
AGENT_CONFIG_PATH=/config/agent-config.yaml

# Integration credentials (encrypted)
SLACK_BOT_TOKEN=xoxb-your-token
SLACK_APP_TOKEN=xapp-your-token
TEAMS_APP_ID=your-app-id
TEAMS_APP_PASSWORD=your-password

# Security
JWT_SECRET=your-secret-key
ENCRYPTION_KEY=your-encryption-key

# Monitoring
PROMETHEUS_ENABLED=true
LOG_LEVEL=INFO
```

### Security Configuration

**secrets.yaml**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: agent-secrets
  namespace: agent-system
type: Opaque
stringData:
  database-url: postgresql://user:pass@postgres:5432/agent_db
  slack-token: xoxb-your-token
  encryption-key: your-encryption-key
```

## Post-Deployment

### 1. Verify Installation
```bash
# Check health endpoint
curl http://agent-service/health

# Expected response:
# {"status": "healthy", "version": "1.0.0"}
```

### 2. Configure Integrations
```bash
# Test Slack integration
curl -X POST http://agent-service/api/integrations/slack/test

# Test Teams integration
curl -X POST http://agent-service/api/integrations/teams/test
```

### 3. Monitor Performance
```bash
# View metrics
curl http://agent-service/metrics

# Access Prometheus dashboard
# Open http://localhost:9090 (or your Prometheus URL)
```

### 4. View Logs
```bash
# Docker Compose
docker-compose logs -f agent

# Kubernetes
kubectl logs -f deployment/agent -n agent-system

# Filter by severity
kubectl logs deployment/agent -n agent-system | grep ERROR
```

## Scaling

### Horizontal Scaling
```bash
# Docker Compose
docker-compose up -d --scale agent=5

# Kubernetes (manual)
kubectl scale deployment agent --replicas=5 -n agent-system

# Kubernetes (auto-scaling)
# HPA will automatically scale based on load
```

### Vertical Scaling
```yaml
# Update resource limits in deployment.yaml
resources:
  requests:
    memory: "2Gi"
    cpu: "1000m"
  limits:
    memory: "4Gi"
    cpu: "2000m"
```

## Monitoring & Observability

### Metrics to Track
- Request latency (p50, p95, p99)
- Throughput (requests per second)
- Error rate
- Resource utilization (CPU, memory)
- Integration health
- Queue depth

### Alerting
```yaml
# prometheus/alerts.yaml
groups:
- name: agent_alerts
  rules:
  - alert: HighErrorRate
    expr: rate(agent_errors_total[5m]) > 0.05
    for: 5m
    annotations:
      summary: "High error rate detected"
  
  - alert: HighLatency
    expr: agent_request_duration_seconds{quantile="0.95"} > 2
    for: 5m
    annotations:
      summary: "High latency detected (p95 > 2s)"
```

## Backup & Recovery

### Database Backup
```bash
# Automated daily backups
0 2 * * * pg_dump -U agent_user agent_db | gzip > /backups/agent_db_$(date +\%Y\%m\%d).sql.gz
```

### Configuration Backup
```bash
# Backup configuration
kubectl get configmap agent-config -n agent-system -o yaml > backup/configmap.yaml
kubectl get secret agent-secrets -n agent-system -o yaml > backup/secrets.yaml
```

## Troubleshooting

### Common Issues

**Issue**: Agent not responding
```bash
# Check if service is running
kubectl get pods -n agent-system
kubectl describe pod <pod-name> -n agent-system

# Check logs
kubectl logs <pod-name> -n agent-system
```

**Issue**: Integration failures
```bash
# Test integration connectivity
kubectl exec -it <pod-name> -n agent-system -- curl https://slack.com/api/auth.test
```

**Issue**: High memory usage
```bash
# Check current usage
kubectl top pods -n agent-system

# Review memory leaks
kubectl logs <pod-name> -n agent-system | grep "memory"
```

## Maintenance

### Updates
```bash
# Update agent version
docker pull your-registry/agent:1.1.0
docker-compose up -d

# Or for Kubernetes
kubectl set image deployment/agent agent=your-registry/agent:1.1.0 -n agent-system
```

### Health Checks
```bash
# Scheduled health check
*/5 * * * * curl -f http://agent-service/health || echo "Agent health check failed"
```

## Security Best Practices

1. **Encryption**: Encrypt all data in transit (TLS) and at rest
2. **Access Control**: Use RBAC for Kubernetes, IAM for cloud
3. **Secrets Management**: Use Vault or cloud secret managers
4. **Network Policies**: Restrict network access between services
5. **Audit Logging**: Enable comprehensive audit trails
6. **Regular Updates**: Keep dependencies and images updated
7. **Vulnerability Scanning**: Regularly scan containers for CVEs

## Support

For issues or questions:
- Documentation: `/docs`
- Issues: GitHub Issues
- Email: support@yourcompany.com
