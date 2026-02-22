# Scaling Guide

Complete guide to horizontal scaling, load balancing, and high availability in ResonantGenesis.

## Overview

ResonantGenesis is designed to scale from small deployments to enterprise-grade infrastructure. This guide covers scaling strategies, load balancing, auto-scaling, and high availability configurations.

## Scaling Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Vertical** | Increase instance size | Quick scaling, single instance |
| **Horizontal** | Add more instances | High availability, large scale |
| **Hybrid** | Combine both | Optimal resource utilization |

## Architecture Overview

```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼────┐        ┌────▼────┐        ┌────▼────┐
    │ API-1   │        │ API-2   │        │ API-3   │
    └────┬────┘        └────┬────┘        └────┬────┘
         │                   │                   │
         └───────────────────┼───────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Message Queue │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼────┐        ┌────▼────┐        ┌────▼────┐
    │Worker-1 │        │Worker-2 │        │Worker-3 │
    └─────────┘        └─────────┘        └─────────┘
```

## Horizontal Scaling

### API Servers

```yaml
# kubernetes/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resonant-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: resonant-api
  template:
    metadata:
      labels:
        app: resonant-api
    spec:
      containers:
      - name: api
        image: resonantgenesis/api:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
```

### Worker Scaling

```yaml
# kubernetes/worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resonant-worker
spec:
  replicas: 5
  selector:
    matchLabels:
      app: resonant-worker
  template:
    metadata:
      labels:
        app: resonant-worker
    spec:
      containers:
      - name: worker
        image: resonantgenesis/worker:latest
        resources:
          requests:
            memory: "1Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "4000m"
```

### Session Scaling

```yaml
# Scale session handlers independently
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resonant-session
spec:
  replicas: 10
  selector:
    matchLabels:
      app: resonant-session
  template:
    spec:
      containers:
      - name: session
        image: resonantgenesis/session:latest
        resources:
          requests:
            memory: "2Gi"
            cpu: "2000m"
```

## Auto-Scaling

### Horizontal Pod Autoscaler

```yaml
# kubernetes/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: resonant-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: resonant-api
  minReplicas: 3
  maxReplicas: 20
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
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 4
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

### Custom Metrics Autoscaling

```yaml
# Scale based on queue depth
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: resonant-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: resonant-worker
  minReplicas: 2
  maxReplicas: 50
  metrics:
  - type: External
    external:
      metric:
        name: queue_depth
        selector:
          matchLabels:
            queue: session-tasks
      target:
        type: AverageValue
        averageValue: 10
```

### AWS Auto Scaling

```python
# Configure AWS Auto Scaling
import boto3

autoscaling = boto3.client('autoscaling')

autoscaling.put_scaling_policy(
    AutoScalingGroupName='resonant-api-asg',
    PolicyName='scale-on-cpu',
    PolicyType='TargetTrackingScaling',
    TargetTrackingConfiguration={
        'PredefinedMetricSpecification': {
            'PredefinedMetricType': 'ASGAverageCPUUtilization'
        },
        'TargetValue': 70.0,
        'ScaleInCooldown': 300,
        'ScaleOutCooldown': 60
    }
)
```

## Load Balancing

### NGINX Configuration

```nginx
# nginx.conf
upstream resonant_api {
    least_conn;
    server api-1:8000 weight=5;
    server api-2:8000 weight=5;
    server api-3:8000 weight=5;
    
    keepalive 32;
}

server {
    listen 80;
    listen 443 ssl http2;
    
    location / {
        proxy_pass http://resonant_api;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 300s;
    }
    
    location /ws {
        proxy_pass http://resonant_api;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### AWS ALB

```yaml
# terraform/alb.tf
resource "aws_lb" "resonant" {
  name               = "resonant-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = var.public_subnets

  enable_deletion_protection = true
}

resource "aws_lb_target_group" "api" {
  name     = "resonant-api-tg"
  port     = 8000
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 3
  }
}
```

### Kubernetes Ingress

```yaml
# kubernetes/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: resonant-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
spec:
  tls:
  - hosts:
    - api.resonantgenesis.xyz
    secretName: tls-secret
  rules:
  - host: api.resonantgenesis.xyz
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: resonant-api
            port:
              number: 8000
```

## Database Scaling

### Read Replicas

```python
# Configure read replicas
from resonantgenesis.db import Database

db = Database(
    primary="postgresql://primary:5432/resonant",
    replicas=[
        "postgresql://replica1:5432/resonant",
        "postgresql://replica2:5432/resonant",
        "postgresql://replica3:5432/resonant"
    ],
    read_strategy="round_robin"
)

# Reads go to replicas
users = await db.read.query("SELECT * FROM users")

# Writes go to primary
await db.write.execute("INSERT INTO users ...")
```

### Connection Pooling

```python
# PgBouncer configuration
# pgbouncer.ini
[databases]
resonant = host=primary port=5432 dbname=resonant

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = md5
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 50
min_pool_size = 10
reserve_pool_size = 10
```

### Database Sharding

```python
# Configure database sharding
from resonantgenesis.db import ShardedDatabase

db = ShardedDatabase(
    shards={
        "shard_0": "postgresql://shard0:5432/resonant",
        "shard_1": "postgresql://shard1:5432/resonant",
        "shard_2": "postgresql://shard2:5432/resonant",
        "shard_3": "postgresql://shard3:5432/resonant"
    },
    shard_key="org_id",
    shard_function="hash_mod"
)

# Queries automatically routed to correct shard
agent = await db.query(
    "SELECT * FROM agents WHERE org_id = %s",
    org_id="org_123"
)
```

## Cache Scaling

### Redis Cluster

```yaml
# kubernetes/redis-cluster.yaml
apiVersion: redis.redis.opstreelabs.in/v1beta1
kind: RedisCluster
metadata:
  name: resonant-redis
spec:
  clusterSize: 6
  kubernetesConfig:
    image: redis:7.0
    resources:
      requests:
        cpu: 500m
        memory: 1Gi
      limits:
        cpu: 2000m
        memory: 4Gi
  storage:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

### Cache Configuration

```python
# Configure distributed cache
from resonantgenesis.cache import DistributedCache

cache = DistributedCache(
    nodes=[
        "redis://cache-1:6379",
        "redis://cache-2:6379",
        "redis://cache-3:6379"
    ],
    cluster_mode=True,
    read_from_replicas=True
)
```

## High Availability

### Multi-Region Deployment

```yaml
# Deploy across regions
regions:
  primary:
    region: us-east-1
    api_replicas: 5
    worker_replicas: 10
    database: primary
    
  secondary:
    region: us-west-2
    api_replicas: 3
    worker_replicas: 5
    database: replica
    
  tertiary:
    region: eu-west-1
    api_replicas: 3
    worker_replicas: 5
    database: replica
```

### Failover Configuration

```python
# Configure automatic failover
from resonantgenesis.ha import FailoverManager

failover = FailoverManager(
    primary_region="us-east-1",
    secondary_regions=["us-west-2", "eu-west-1"],
    health_check_interval=10,
    failover_threshold=3,
    auto_failback=True,
    failback_delay=300
)

# Monitor and failover
await failover.start()
```

### Health Checks

```python
# Comprehensive health checks
from resonantgenesis.health import HealthChecker

health = HealthChecker()

@health.check("database")
async def check_database():
    return await db.ping()

@health.check("cache")
async def check_cache():
    return await cache.ping()

@health.check("queue")
async def check_queue():
    return await queue.ping()

# Expose health endpoint
@app.get("/health")
async def health_endpoint():
    return await health.run_all()
```

## Queue Scaling

### RabbitMQ Cluster

```yaml
# kubernetes/rabbitmq.yaml
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: resonant-rabbitmq
spec:
  replicas: 3
  resources:
    requests:
      cpu: 500m
      memory: 1Gi
    limits:
      cpu: 2000m
      memory: 4Gi
  persistence:
    storageClassName: fast-ssd
    storage: 50Gi
```

### Queue Partitioning

```python
# Partition queues for scaling
from resonantgenesis.queue import PartitionedQueue

queue = PartitionedQueue(
    name="session-tasks",
    partitions=16,
    partition_key="agent_id"
)

# Messages distributed across partitions
await queue.publish(
    task={"goal": "Process data"},
    agent_id="agent_123"  # Determines partition
)
```

## Performance Optimization

### Connection Pooling

```python
# Optimize connection pools
from resonantgenesis.pool import ConnectionPool

pool = ConnectionPool(
    min_size=10,
    max_size=100,
    max_idle_time=300,
    max_lifetime=3600,
    health_check_interval=30
)
```

### Request Batching

```python
# Batch requests for efficiency
from resonantgenesis.batch import BatchProcessor

batch = BatchProcessor(
    max_batch_size=100,
    max_wait_time=0.1,  # 100ms
    processor=process_batch
)

# Requests automatically batched
result = await batch.submit(request)
```

### Async Processing

```python
# Use async for I/O operations
import asyncio

async def process_requests(requests):
    # Process concurrently
    tasks = [process_request(r) for r in requests]
    results = await asyncio.gather(*tasks)
    return results
```

## Monitoring at Scale

### Metrics Collection

```python
# Prometheus metrics
from prometheus_client import Counter, Histogram, Gauge

requests_total = Counter(
    'requests_total',
    'Total requests',
    ['method', 'endpoint', 'status']
)

request_duration = Histogram(
    'request_duration_seconds',
    'Request duration',
    ['method', 'endpoint']
)

active_connections = Gauge(
    'active_connections',
    'Active connections',
    ['service']
)
```

### Scaling Alerts

```yaml
# prometheus/alerts.yaml
groups:
- name: scaling
  rules:
  - alert: HighCPUUsage
    expr: avg(cpu_usage) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
      
  - alert: HighMemoryUsage
    expr: avg(memory_usage) > 85
    for: 5m
    labels:
      severity: warning
      
  - alert: HighQueueDepth
    expr: queue_depth > 1000
    for: 2m
    labels:
      severity: critical
```

## Best Practices

### For Scaling

1. **Start horizontal** - Scale out before up
2. **Use auto-scaling** - Respond to demand
3. **Monitor metrics** - Track scaling triggers
4. **Test at scale** - Load test regularly
5. **Plan capacity** - Anticipate growth

### For High Availability

1. **Multi-region** - Geographic redundancy
2. **Auto-failover** - Automatic recovery
3. **Health checks** - Detect failures fast
4. **Graceful degradation** - Partial functionality
5. **Regular drills** - Test failover procedures

### For Performance

1. **Connection pooling** - Reuse connections
2. **Caching** - Reduce database load
3. **Async processing** - Non-blocking I/O
4. **Batch operations** - Reduce overhead
5. **Profile regularly** - Find bottlenecks

## API Reference

### Get Scaling Status

```bash
GET /api/v1/scaling/status
```

### Trigger Scale Event

```bash
POST /api/v1/scaling/scale
```

### Get Health Status

```bash
GET /api/v1/health
```

### Get Metrics

```bash
GET /api/v1/metrics
```

---

**Need scaling help?** Contact infrastructure@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
