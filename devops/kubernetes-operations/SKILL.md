---
name: kubernetes-operations
description: 
category: devops
tags: [kubernetes-operations]
---

## When to Use
Deploy, manage, and troubleshoot workloads on Kubernetes. Covers deployments, services, ingress, secrets, resource management, and cluster operations with kubectl and Helm.

## Core Concepts
- **Declarative state**: Apply YAML manifests; kubectl reconciles desired vs actual state
- **Namespaces**: Logical isolation for teams/environments
- **Services**: ClusterIP (internal), NodePort, LoadBalancer, ExternalName
- **Ingress**: HTTP routing with TLS termination (nginx, traefik, ALB)
- **HPA**: Horizontal Pod Autoscaler scales based on CPU/memory/custom metrics
- **RBAC**: Role-based access control for API permissions

## Workflow
1. Define Deployment, Service, ConfigMap, Secret in YAML
2. Apply with `kubectl apply -f manifests/`
3. Verify rollout: `kubectl rollout status deployment/app`
4. Configure Ingress for external access
5. Set up HPA for autoscaling
6. Monitor with `kubectl top pods` and Prometheus metrics

## Key Patterns
```yaml
# Deployment with rolling update and resource limits
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: api-server
  template:
    metadata:
      labels:
        app: api-server
    spec:
      containers:
      - name: api
        image: registry.example.com/api:1.2.3
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "1"
            memory: "512Mi"
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
```

```yaml
# HPA autoscaling
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 2
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

```bash
# Essential kubectl commands
kubectl apply -f manifests/
kubectl rollout status deployment/api-server
kubectl rollout undo deployment/api-server          # rollback
kubectl rollout history deployment/api-server       # see history
kubectl top pods -n production                      # resource usage
kubectl logs -f deployment/api-server --tail=100    # live logs
kubectl exec -it <pod> -- /bin/sh                   # debug shell
kubectl get events -n production --sort-by=.lastTimestamp
```

## Pitfalls
- **Resource requests without limits**: Pods can OOM-kill or starve nodes
- **Missing readiness probes**: Traffic sent to unready pods causes 502s
- **Rolling update with maxUnavailable > 0**: Can reduce capacity during deploys
- **Secrets in plaintext**: Use sealed-secrets or external-secrets-operator
- **Node affinity drift**: Pod scheduling changes on node failure
- **Image pull secrets**: Private registries need imagePullSecrets in spec

## Verification
```bash
# Verify deployment health
kubectl get pods -n production -o wide
kubectl describe deployment api-server -n production

# Verify services have endpoints
kubectl get endpoints api-server -n production

# Verify ingress is routing
kubectl get ingress -n production
curl -H "Host: api.example.com" http://INGRESS_IP/healthz

# Verify autoscaling is configured
kubectl get hpa -n production
```