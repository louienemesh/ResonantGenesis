# Kubernetes Deployment Guide

Complete guide to Kubernetes deployment, Helm charts, and container orchestration for ResonantGenesis.

## Overview

ResonantGenesis is designed for cloud-native deployment on Kubernetes. This guide covers deployment strategies, Helm charts, configuration, and best practices for running ResonantGenesis on Kubernetes.

## Prerequisites

| Requirement | Version |
|-------------|---------|
| Kubernetes | 1.25+ |
| Helm | 3.10+ |
| kubectl | 1.25+ |
| Container Runtime | Docker/containerd |

## Quick Start

### Install with Helm

```bash
# Add Helm repository
helm repo add resonant https://charts.resonantgenesis.xyz
helm repo update

# Install ResonantGenesis
helm install resonant resonant/resonantgenesis \
  --namespace resonant \
  --create-namespace \
  --set api.apiKey=your_api_key
```

### Verify Installation

```bash
# Check pods
kubectl get pods -n resonant

# Check services
kubectl get svc -n resonant

# Check ingress
kubectl get ingress -n resonant
```

## Helm Chart

### Chart Structure

```
resonantgenesis/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── api-deployment.yaml
│   ├── api-service.yaml
│   ├── worker-deployment.yaml
│   ├── session-deployment.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   └── serviceaccount.yaml
└── charts/
    ├── postgresql/
    ├── redis/
    └── rabbitmq/
```

### values.yaml

```yaml
# values.yaml
global:
  imageRegistry: ""
  imagePullSecrets: []

# API Server
api:
  replicaCount: 3
  image:
    repository: resonantgenesis/api
    tag: "1.0.0"
    pullPolicy: IfNotPresent
  
  resources:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "2Gi"
      cpu: "2000m"
  
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 20
    targetCPUUtilization: 70
    targetMemoryUtilization: 80
  
  service:
    type: ClusterIP
    port: 8000

# Worker
worker:
  replicaCount: 5
  image:
    repository: resonantgenesis/worker
    tag: "1.0.0"
  
  resources:
    requests:
      memory: "1Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "4000m"
  
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 50

# Session Handler
session:
  replicaCount: 10
  image:
    repository: resonantgenesis/session
    tag: "1.0.0"
  
  resources:
    requests:
      memory: "2Gi"
      cpu: "2000m"
    limits:
      memory: "8Gi"
      cpu: "8000m"

# Ingress
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: api.resonantgenesis.xyz
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: resonant-tls
      hosts:
        - api.resonantgenesis.xyz

# Database
postgresql:
  enabled: true
  auth:
    postgresPassword: ""
    database: resonant
  primary:
    persistence:
      size: 100Gi
  readReplicas:
    replicaCount: 2

# Cache
redis:
  enabled: true
  architecture: replication
  auth:
    enabled: true
  master:
    persistence:
      size: 10Gi
  replica:
    replicaCount: 2

# Message Queue
rabbitmq:
  enabled: true
  auth:
    username: resonant
    password: ""
  clustering:
    enabled: true
  replicaCount: 3
  persistence:
    size: 20Gi
```

## Deployment Configurations

### API Deployment

```yaml
# templates/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "resonant.fullname" . }}-api
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
    app.kubernetes.io/component: api
spec:
  {{- if not .Values.api.autoscaling.enabled }}
  replicas: {{ .Values.api.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "resonant.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: api
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
      labels:
        {{- include "resonant.selectorLabels" . | nindent 8 }}
        app.kubernetes.io/component: api
    spec:
      serviceAccountName: {{ include "resonant.serviceAccountName" . }}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      containers:
        - name: api
          image: "{{ .Values.api.image.repository }}:{{ .Values.api.image.tag }}"
          imagePullPolicy: {{ .Values.api.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            {{- toYaml .Values.api.resources | nindent 12 }}
          envFrom:
            - configMapRef:
                name: {{ include "resonant.fullname" . }}-config
            - secretRef:
                name: {{ include "resonant.fullname" . }}-secrets
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    {{- include "resonant.selectorLabels" . | nindent 20 }}
                    app.kubernetes.io/component: api
                topologyKey: kubernetes.io/hostname
```

### Worker Deployment

```yaml
# templates/worker-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "resonant.fullname" . }}-worker
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
    app.kubernetes.io/component: worker
spec:
  replicas: {{ .Values.worker.replicaCount }}
  selector:
    matchLabels:
      {{- include "resonant.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: worker
  template:
    metadata:
      labels:
        {{- include "resonant.selectorLabels" . | nindent 8 }}
        app.kubernetes.io/component: worker
    spec:
      containers:
        - name: worker
          image: "{{ .Values.worker.image.repository }}:{{ .Values.worker.image.tag }}"
          command: ["python", "-m", "resonant.worker"]
          resources:
            {{- toYaml .Values.worker.resources | nindent 12 }}
          envFrom:
            - configMapRef:
                name: {{ include "resonant.fullname" . }}-config
            - secretRef:
                name: {{ include "resonant.fullname" . }}-secrets
```

### Session Deployment

```yaml
# templates/session-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "resonant.fullname" . }}-session
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
    app.kubernetes.io/component: session
spec:
  replicas: {{ .Values.session.replicaCount }}
  selector:
    matchLabels:
      {{- include "resonant.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: session
  template:
    metadata:
      labels:
        {{- include "resonant.selectorLabels" . | nindent 8 }}
        app.kubernetes.io/component: session
    spec:
      containers:
        - name: session
          image: "{{ .Values.session.image.repository }}:{{ .Values.session.image.tag }}"
          command: ["python", "-m", "resonant.session_handler"]
          resources:
            {{- toYaml .Values.session.resources | nindent 12 }}
          envFrom:
            - configMapRef:
                name: {{ include "resonant.fullname" . }}-config
            - secretRef:
                name: {{ include "resonant.fullname" . }}-secrets
```

## ConfigMaps and Secrets

### ConfigMap

```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "resonant.fullname" . }}-config
data:
  LOG_LEVEL: {{ .Values.config.logLevel | quote }}
  LOG_FORMAT: {{ .Values.config.logFormat | quote }}
  ENVIRONMENT: {{ .Values.config.environment | quote }}
  
  # Database
  DATABASE_HOST: {{ include "resonant.postgresql.host" . | quote }}
  DATABASE_PORT: "5432"
  DATABASE_NAME: {{ .Values.postgresql.auth.database | quote }}
  
  # Redis
  REDIS_HOST: {{ include "resonant.redis.host" . | quote }}
  REDIS_PORT: "6379"
  
  # RabbitMQ
  RABBITMQ_HOST: {{ include "resonant.rabbitmq.host" . | quote }}
  RABBITMQ_PORT: "5672"
```

### Secrets

```yaml
# templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "resonant.fullname" . }}-secrets
type: Opaque
data:
  API_KEY: {{ .Values.api.apiKey | b64enc | quote }}
  DATABASE_PASSWORD: {{ .Values.postgresql.auth.postgresPassword | b64enc | quote }}
  REDIS_PASSWORD: {{ .Values.redis.auth.password | b64enc | quote }}
  RABBITMQ_PASSWORD: {{ .Values.rabbitmq.auth.password | b64enc | quote }}
```

## Ingress Configuration

### NGINX Ingress

```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "resonant.fullname" . }}
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    nginx.ingress.kubernetes.io/websocket-services: {{ include "resonant.fullname" . }}-api
    {{- with .Values.ingress.annotations }}
    {{- toYaml . | nindent 4 }}
    {{- end }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "resonant.fullname" $ }}-api
                port:
                  number: 8000
          {{- end }}
    {{- end }}
{{- end }}
```

## Auto-Scaling

### Horizontal Pod Autoscaler

```yaml
# templates/hpa.yaml
{{- if .Values.api.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "resonant.fullname" . }}-api
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "resonant.fullname" . }}-api
  minReplicas: {{ .Values.api.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.api.autoscaling.maxReplicas }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.api.autoscaling.targetCPUUtilization }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.api.autoscaling.targetMemoryUtilization }}
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
{{- end }}
```

### KEDA Scaling

```yaml
# keda-scaledobject.yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: resonant-worker-scaler
spec:
  scaleTargetRef:
    name: resonant-worker
  minReplicaCount: 2
  maxReplicaCount: 50
  triggers:
    - type: rabbitmq
      metadata:
        host: amqp://resonant:password@rabbitmq:5672/
        queueName: session-tasks
        queueLength: "10"
```

## Pod Disruption Budget

```yaml
# templates/pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "resonant.fullname" . }}-api
spec:
  minAvailable: 2
  selector:
    matchLabels:
      {{- include "resonant.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: api
```

## Service Account

```yaml
# templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "resonant.serviceAccountName" . }}
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
  annotations:
    {{- with .Values.serviceAccount.annotations }}
    {{- toYaml . | nindent 4 }}
    {{- end }}
```

## Network Policies

```yaml
# templates/networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "resonant.fullname" . }}-api
spec:
  podSelector:
    matchLabels:
      {{- include "resonant.selectorLabels" . | nindent 6 }}
      app.kubernetes.io/component: api
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8000
  egress:
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: postgresql
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: redis
      ports:
        - protocol: TCP
          port: 6379
```

## Monitoring

### ServiceMonitor

```yaml
# templates/servicemonitor.yaml
{{- if .Values.metrics.serviceMonitor.enabled }}
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "resonant.fullname" . }}
  labels:
    {{- include "resonant.labels" . | nindent 4 }}
spec:
  selector:
    matchLabels:
      {{- include "resonant.selectorLabels" . | nindent 6 }}
  endpoints:
    - port: http
      path: /metrics
      interval: 30s
{{- end }}
```

## Helm Commands

### Install

```bash
# Install with custom values
helm install resonant resonant/resonantgenesis \
  -f custom-values.yaml \
  --namespace resonant \
  --create-namespace
```

### Upgrade

```bash
# Upgrade release
helm upgrade resonant resonant/resonantgenesis \
  -f custom-values.yaml \
  --namespace resonant
```

### Rollback

```bash
# Rollback to previous version
helm rollback resonant 1 --namespace resonant
```

### Uninstall

```bash
# Uninstall release
helm uninstall resonant --namespace resonant
```

## Best Practices

### For Production

1. **Use resource limits** - Prevent resource exhaustion
2. **Enable HPA** - Auto-scale based on load
3. **Configure PDB** - Ensure availability during updates
4. **Use network policies** - Restrict traffic
5. **Enable monitoring** - Track metrics and logs

### For Security

1. **Use secrets** - Never hardcode credentials
2. **Run as non-root** - Security context
3. **Use service accounts** - RBAC permissions
4. **Enable TLS** - Encrypt traffic
5. **Scan images** - Vulnerability scanning

### For Reliability

1. **Multi-zone deployment** - Geographic redundancy
2. **Pod anti-affinity** - Spread across nodes
3. **Liveness/readiness probes** - Health checks
4. **Graceful shutdown** - Handle SIGTERM
5. **Resource requests** - Scheduling guarantees

## Troubleshooting

### Check Pod Status

```bash
kubectl get pods -n resonant
kubectl describe pod <pod-name> -n resonant
kubectl logs <pod-name> -n resonant
```

### Check Events

```bash
kubectl get events -n resonant --sort-by='.lastTimestamp'
```

### Debug Networking

```bash
kubectl run debug --rm -it --image=nicolaka/netshoot -- /bin/bash
```

---

**Need Kubernetes help?** Contact infrastructure@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
