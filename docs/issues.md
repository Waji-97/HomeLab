# Collection of Issues
Short collection of issues faced while deploying different components in the cluster.

## Prometheus Stack
- Need to have `Replace=true` option or ArgoCD will keep showing `SyncFailed`. The issue was faced due to long last-applied annotation that comes with `kubectl apply`. The `Replace=true` option will replace `kubectl apply` with `kubectl replace`
- admissionwebhook-create keeps crashlooping due to forbidden error (not sufficient permissions to get secrets?). Disabled