# flux
Kubernetes cluster state managed by flux.
Single cluster, installing storage first, then infrastructure, then apps.

## Quick Start
+ Install Flux Operator via kubectl:
  `kubectl apply -f https://github.com/controlplaneio-fluxcd/flux-operator/releases/latest/download/install.yaml`
+ Create FluxInstance:
  `kubectl apply -f infra/controllers/flux-system/flux-instance.yaml`
+ Configure flux:
  `kubectl apply -k infra/configs/flux-system/`
+ Once flux is running, flux-operator will be managed by helm chart under `infra/controllers/flux-system/`

## Directory Structure
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

## Encryption
+ For flux Kustomize controller:
  + Secret `flux-sops`, key `sops.agekey`: **privkey** for decrypt
+ For CLI `sops`:
  + `.sops.yaml`: **pubkey** for encrypt
  + `~/.config/sops/age/keys.txt`: **privkey** for decrypt
