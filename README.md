# flux
Kubernetes cluster state managed by flux.
Single cluster, installing storage first, then infrastructure, then apps.

## Quick Start
`kubectl apply -k .`

## Directory Structure
+ `.sops.yaml`: **pubkey** for CLI encryption (privkey is in ~/.config/sops/age/keys.txt)
+ `kustomize.yaml`: points to all **flux** Kustomizations
+ `apps/`:
  + `flux.yaml`: flux Kustomization: **decrypts** and points to `kustomization.yaml`
  + `kustomization.yaml`: points to app **subdirs**
  + `$APP/`:
    + `kustomization.yaml`:
      + Sets **namespace**
      + Loads **resources** in current dir
      + Overrides **image** tag
    + `src.yaml`: git or OCI **repo** for helm chart
    + `helm.yaml`: loads **helm** chart, referencing ConfigMap for values
    + `values.yaml`: **ConfigMap** for helm values
    + `secret.yaml`: encrypted **secrets** (if needed)
  + infra/controllers/: define CRDs for infrastructure used by all clusters
    + flux.yaml: use repo source, decrypt, point to kustomization.yaml
    + kustomization.yaml: load resources
  + infra/configs/: same for resources using above CRDs
  + storage/controllers/: same for storage system (e.g., rook-ceph)
  + storage/configs/: same for resources using above CRDs
