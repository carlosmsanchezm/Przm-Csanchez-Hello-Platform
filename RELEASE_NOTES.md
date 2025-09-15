## What's in this release
- Multi-arch (linux/amd64, linux/arm64) image published to private GHCR
- Runtime hardening: non-root, read-only root FS + tmpfs /tmp
- Health probes, resource limits, NetworkPolicy default-deny

## Artifact (immutable)
`ghcr.io/carlosmsanchezm/hello-app@sha256:e50def02d916eee491c1f38de61ed599d01e34024fcf7377a2e697f22f3fc241`

### Local quick start
```bash
echo "$GH_TOKEN" | docker login ghcr.io -u carlosmsanchezm --password-stdin
docker pull ghcr.io/carlosmsanchezm/hello-app@sha256:e50def02d916eee491c1f38de61ed599d01e34024fcf7377a2e697f22f3fc241
docker run --rm -p 8080:8080 --read-only --tmpfs /tmp:rw,size=64m ghcr.io/carlosmsanchezm/hello-app@sha256:e50def02d916eee491c1f38de61ed599d01e34024fcf7377a2e697f22f3fc241
```

### Helm values for platform

```yaml
image:
  repository: ghcr.io/carlosmsanchezm/hello-app
  digest: sha256:e50def02d916eee491c1f38de61ed599d01e34024fcf7377a2e697f22f3fc241
```