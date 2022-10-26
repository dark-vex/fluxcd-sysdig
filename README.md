# fluxcd-sysdig

### Info
This repository is based on the great tutorial example prepared by [fluxcd team](https://github.com/fluxcd/flux2-kustomize-helm-example) and show how-to deploy and manage sysdig-agent and sysdig-admission-controller chart via FluxCD

### Repository structure

The Git repository contains the following top directories:

- **apps** dir contains Helm releases with a common configuration for all Kubernetes clusters and for each cluster a specific custom configuration
- **charts** dir contains Helm chart repository definitions
- **clusters** dir contains the Flux configuration per cluster
```
├── apps
│   ├── common
│   ├── dev-1
│   ├── dev-2
│   ├── prod-1
│   └── prod-2
├── charts
│   ├── sysdig.yaml
├── clusters
│   ├── dev-1
│   ├── dev-2
│   ├── prod-1
│   └── prod-2
```

The apps configuration is structured into:

- **apps/common/** dir contains namespaces, Helm release definitions with sysdig-agent and sysdig-admission-controller common configurations
- **apps/prod-1/** dir contains prod-1 cluster values
- **apps/prod-2/** dir contains prod-2 cluster values
- **apps/dev-1/** dir contains dev-1 cluster values
- **apps/dev-2/** dir contains dev-2 cluster values

The idea is to have a base common settings for Sysdig components, like: collector address/port, access-key, etc.. and then for each cluster change specific Sysdig settings, like for example the cluster name or resources assigned to the agents

**Note** This example cover the usage of external secrets but it doesn't cover how-to create them, there are seveal methods and tools for managing kubernetes secret so I don't cover this topic

```
./apps/
├── common
│   ├── sysdig-admission-controller
│   │   ├── kustomization.yaml
│   │   ├── namespace.yaml
│   │   └── release.yaml
│   └── sysdig-agent
│       ├── kustomization.yaml
│       ├── namespace.yaml
│       └── release.yaml
├── prod-1
│   ├── sysdig-admission-controller
│   │   └── release.yaml
│   ├── sysdig-agent
│   │   └── release.yaml
│   └── kustomization.yaml
├── prod-2
│   ├── sysdig-admission-controller
│   │   └── release.yaml
│   ├── sysdig-agent
│   │   └── release.yaml
│   └── kustomization.yaml
├── dev-1
│   ├── sysdig-admission-controller
│   │   └── release.yaml
│   ├── sysdig-agent
│   │   └── release.yaml
│   └── kustomization.yaml
└── dev-2
│   ├── sysdig-admission-controller
│   │   └── release.yaml
│   ├── sysdig-agent
│   │   └── release.yaml
│   └── kustomization.yaml
```

To confirm that your config files changes are correct before pushing the them to the repo, you can run from the root folder of the repo:
```
$ kustomize build apps/dev-1

# Output
apiVersion: v1
kind: Namespace
metadata:
  name: sysdig-agent
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: sysdig-deploy
  namespace: sysdig-agent
spec:
  chart:
    spec:
      chart: sysdig-deploy
      interval: 5m
      sourceRef:
        kind: HelmRepository
        name: sysdig-charts
        namespace: flux-system
      version: '>=1.3.20'
  install:
    createNamespace: true
    remediation:
      retries: 5
  interval: 5m
  upgrade:
    remediation:
      retries: 5
  values:
    agent:
      enabled: true
      resourceProfile: custom
      resources:
        limits:
          cpu: 3
          memory: 3Gi
        requests:
          cpu: 3
          memory: 3Gi
      sysdig:
        settings:
          app_checks_enabled: false
          feature:
            mode: monitor
          jmx:
            enabled: false
          prometheus:
            enabled: true
            prom_service_discovery: true
    global:
      clusterConfig:
        name: dev-1
      image:
        registry: quay.io
      sysdig:
        accessKeySecret: sysdig-agent
        region: eu1
    nodeAnalyzer:
      enabled: true
    rapidResponse:
      enabled: false
```
