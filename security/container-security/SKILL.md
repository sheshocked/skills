---
name: container-security
description: 
category: security
tags: [container-security]
---

## When to Use
Use this skill when securing containerized environments — Docker image scanning, Kubernetes RBAC, runtime security with seccomp/AppArmor, network policies, and secrets management in containers.

## Core Concepts
- **Image scanning**: Detect CVEs in base images and dependencies
- **Least privilege**: Non-root containers, read-only filesystems, drop capabilities
- **Runtime security**: seccomp profiles, AppArmor/SELinux enforcement
- **Network policies**: Kubernetes NetworkPolicy for pod-to-pod communication control
- **RBAC**: Kubernetes Role/ClusterRole for access control
- **Supply chain**: Image signing (Cosign), provenance verification (SLSA)

## Workflow
1. **Image scan**: Scan base images and build artifacts for CVEs
2. **Dockerfile hardening**: Use distroless/Alpine, multi-stage builds, non-root user
3. **Kubernetes RBAC**: Create minimal roles, audit cluster bindings
4. **Network policies**: Implement default-deny, allow only necessary pod communication
5. **Runtime security**: Apply seccomp and AppArmor profiles
6. **Secrets management**: Use external secrets operators, not Kubernetes Secrets
7. **Admission control**: OPA/Gatekeeper policies for pod security standards
8. **Image signing**: Sign images with Cosign, verify at deployment

## Key Patterns

### Docker Image Hardening
```dockerfile
# Secure Dockerfile — multi-stage, non-root, minimal
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server

# Use distroless for runtime
FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /app/server /server
USER nonroot:nonroot
EXPOSE 8080
ENTRYPOINT ["/server"]
```

### Image Scanning with Trivy
```bash
# Scan image for vulnerabilities
trivy image myapp:latest --severity HIGH,CRITICAL --exit-code 1

# Scan with SBOM generation
trivy image --format spdx-json myapp:latest > sbom.json

# Scan filesystem
trivy fs --security-checks vuln,secret,misconfig .

# CI/CD integration — fail on critical CVEs
trivy image myapp:latest --exit-code 1 --severity CRITICAL

# Fix base image
trivy image --ignore-unfixed myapp:latest
```

### Kubernetes RBAC Setup
```yaml
# Minimal namespace-scoped role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: app-deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "update", "patch"]
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]

---
# Bind role to service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-deployer-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: deployer-sa
  namespace: production
roleRef:
  kind: Role
  name: app-deployer
  apiGroup: rbac.authorization.k8s.io
```

### Kubernetes Network Policies
```yaml
# Default deny all ingress in namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress

---
# Allow frontend → backend on port 8080
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080

---
# Allow backend → database on port 5432
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-db
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - protocol: TCP
      port: 5432
```

### Pod Security Standards (Restricted)
```yaml
# Security context for restricted pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        runAsGroup: 65534
        fsGroup: 65534
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: myapp:latest@sha256:abc123...
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop: ["ALL"]
          runAsNonRoot: true
        resources:
          limits:
            cpu: "500m"
            memory: "256Mi"
          requests:
            cpu: "100m"
            memory: "128Mi"
        volumeMounts:
        - name: tmp
          mountPath: /tmp
      volumes:
      - name: tmp
        emptyDir: {}
```

### OPA Gatekeeper Policy
```yaml
# Require image digests (no mutable tags)
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sAllowedRepos
metadata:
  name: require-digest
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    namespaces: ["production"]
  parameters:
    repos:
    - "myregistry.com/"
    - "gcr.io/distroless/"

# Deny privileged containers
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sPSPPrivilegedContainer
metadata:
  name: deny-privileged
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
    namespaces: ["production"]

# Require resource limits
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sContainerLimits
metadata:
  name: require-limits
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
  parameters:
    cpu: "2"
    memory: "2Gi"
```

## Pitfalls
- **Root containers**: Running as root in containers is dangerous — always set `runAsNonRoot: true`
- **Mutable tags**: `image:latest` can change without notice — use SHA256 digests for production
- **Secrets in env vars**: Kubernetes Secrets are base64-encoded, not encrypted — use external secrets operator
- **Network policy gaps**: No default-deny means all pod-to-pod traffic is allowed
- **Docker socket**: Mounting `/var/docker.sock` gives container full host control — avoid
- **Image pulling**: Use private registries with image pull secrets — never pull from public registries in production

## Verification
- `trivy image myapp:latest --exit-code 1` — no HIGH/CRITICAL CVEs
- `kubectl auth can-i --list --as=system:serviceaccount:prod:deployer` — minimal permissions
- `kubectl get networkpolicies -n production` — default-deny and allow policies exist
- `kubectl get pods -n production -o json | jq '.items[].spec.containers[].securityContext'` — non-root, read-only
- `cosign verify myapp@sha256:abc123...` — image signature valid
- `kubectl get clusterrolebindings` — no overly broad cluster-admin bindings