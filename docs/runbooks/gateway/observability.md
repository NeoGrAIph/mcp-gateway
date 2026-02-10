# Observability (Kubernetes)

Status: active | Audience: support | Last reviewed: 2026-02-09

## Goal

Quickly verify gateway pods are running and fetch logs.

## Preconditions

- You have access to a Kubernetes cluster where gateway is deployed.
- Namespace is `adapter` (default in provided manifests).

## Procedure

1. List pods:
   - `kubectl get pods -n adapter`
2. Fetch logs (label selector):
   - `kubectl logs -n adapter -l app=mcpgateway`

## Verification

- Pods are `Running` and logs contain no crash loops.

## References

- Local k8s manifest (labels): `deployment/k8s/local-deployment.yml`
- Deployment guide: `deployment/infra/README.md`
