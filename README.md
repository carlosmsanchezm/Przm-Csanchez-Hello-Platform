# hello-platform

Decisive defaults for production:
- **Immutable image** (digest pinned) ✅ `sha256:075a028ea47969d41f577ea09fa7c76d5fcc61ea261a9a1b7b6512212e3be340`
- **NetworkPolicy** on by default (ingress allow-list + DNS-only egress)
- **No service account token**, non-root, read-only FS, dropped caps
- **PDB** enabled, spread across nodes
- **tmpfs volumes** for writable directories (read-only root FS compatible)

## Configure

### Image Settings
Current pinned image in `helm/hello/values.yaml`:
```yaml
image:
  repository: ghcr.io/carlosmsanchezm/hello-app
  digest: sha256:075a028ea47969d41f577ea09fa7c76d5fcc61ea261a9a1b7b6512212e3be340
  pullPolicy: IfNotPresent
imagePullSecrets: ["ghcr-creds"]  # Required for private GHCR
```

### Private Registry Setup
**GHCR is configured as private.** Create pull secret:
```bash
export GH_USER="carlosmsanchezm"
export GH_TOKEN="your-classic-pat-with-read:packages"

kubectl create secret docker-registry ghcr-creds \
  --docker-server=ghcr.io \
  --docker-username="$GH_USER" \
  --docker-password="$GH_TOKEN" \
  -n default
```

## Deploy
```bash
helm upgrade --install hello ./helm/hello -n default
kubectl rollout status deploy/hello -n default --timeout=60s
kubectl port-forward svc/hello 8080:80 -n default &
curl -s http://localhost:8080/; echo
curl -s http://localhost:8080/healthz; echo
```

## Operational Notes

### Security Posture Validated ✅
- **Authentication**: Private GHCR image pull with `imagePullSecrets`
- **Non-root runtime**: User 10001:10001, no privilege escalation
- **Read-only filesystem**: Root FS read-only + tmpfs `/tmp` (64Mi limit)
- **Network isolation**: NetworkPolicy allows ingress from `role=ingress` only, DNS-only egress
- **Resource limits**: CPU 500m/50m, Memory 256Mi/64Mi
- **High availability**: PDB minAvailable=1, topology spread constraints

### Platform Compatibility
- **Multi-arch images**: linux/amd64 + linux/arm64 ✅ (manifest list auto-selects architecture)
- **Apple Silicon**: Native ARM64 performance (no emulation warnings)
- **Intel/AMD**: Native AMD64 performance
- **Windows ARM**: Compatible via ARM64 variant
- **Kubernetes**: Tested on kind/Docker Desktop
- **Required volumes**: tmpfs for `/tmp` (Python requires writable temp directory)

### Performance Benefits
- **Native execution**: No platform mismatch warnings
- **Faster startup**: Architecture-specific optimizations
- **Universal compatibility**: Same image reference works across all platforms

### Monitoring Deployment
```bash
# Watch deployment
kubectl get pods -l app.kubernetes.io/name=hello -w

# Check pod events if issues
kubectl describe pod -l app.kubernetes.io/name=hello | sed -n '/Events:/,$p'

# View container logs
kubectl logs -l app.kubernetes.io/name=hello --tail=20
```