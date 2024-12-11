# flux
Kubernetes cluster state managed by flux

## Quick Start
+ Tell flux to use this repo: `kubectl apply -f flux-src.yaml`
+ Setup cluster: `kubectl apply -k clusters/$CLUSTER`

## Directory Structure
+ clusters/$CLUSTER/: cluster-specific config
  + .sops.yaml: pubkey for CLI encryption (privkey is in ~/.config/sops/age/keys.txt)
  + kustomize.yaml: points to all flux Kustomizations
  + apps/
    + flux.yaml: flux Kustomization:
      + Depends on infra-configs flux Kustomization
      + Uses repo source
      + Applies SOPS decryption
      + Points to kustomization.yaml
    + kustomization.yaml:
      + Points to app subdirs
    + $APP/
      + kustomization.yaml:
        + Sets namespace
        + Points to /apps/$APP as a base resource
        + Applies patches (including encrypted content)
          + Override helm values via ConfigMap
        + Overrides image tags
      + (patch files)
  + infra/controllers/: define CRDs for infrastructure used by all clusters
    + flux.yaml: use repo source, decrypt, point to kustomization.yaml
    + kustomization.yaml: apply patches and additional cluster-specific resources for controllers
  + infra/configs/: same for CRDs using above definitions
  + storage/controllers/: same for storage system (e.g., rook-ceph)
  + storage/configs/
+ apps/$APP/: cluster-agnostic
  + No kustomization.yaml needed (import all files)
  + Define source repo
  + Create HelmRelease (with valuesFrom ConfigMap)
  + General settings in ConfigMap
+ infra/controllers/
+ infra/configs/
+ storage/controllers/
+ storage/configs/

## Dependencies
+ Sequencing is achieved via dependencies in the flux Kustomizations
+ Within kustomization.yaml, resources are applied in order
  (alphabetical order by filename, if kustomiztion.yaml is auto-generated)
