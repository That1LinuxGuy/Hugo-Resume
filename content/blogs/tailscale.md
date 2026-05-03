+++
date = '2026-05-02'
draft = false
title = 'Tailscale Kubernetes Operator'
tags = ["networking", "kubernetes"]
+++

# Setting up Taislcale K8S Operator

As clear as the docs are, I ran into significant errors running this on my homelab. The conbimation of fluxCD, tailscale ACL hyper-specificity, and SOPS secrets made for quite a long process.

## Why do I need it?

In my case, I wanted to have remote access to admin data without expoisng those servives directly to the internet. Thankfully, not I have access the the longhorn GUI and Grafana with tailscale, but these are not exposed anywhere on the public web

## How did I do it?

Overall, it was very simple - but there were a few tricks that made it much quicker when using fluxCD to manage the config.

First and foremost, you have to add the Owners to your tailscale ACL file:

``` 
"tagOwners": {
  "tag:k8s-operator": [your-email@provider.com],
  "tag:k8s": ["tag:k8s-operator"],
}
```

Both k8s-operator and k8s are required: 

From my understanding, k8s-operator is used by by the bootstrap pod to connect your local machines to the tailscale netowrk and the k8s tag is used by the services you create with ingress/egress routes


``` YAML
apiVersion: v1
kind: Namespace
metadata:
  name: tailscale
```

The namespace has to come first, after that:

``` YAML
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: tailscale
  namespace: tailscale
spec:
  interval: 1h
  url: https://pkgs.tailscale.com/helmcharts
```

``` YAML
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: tailscale-operator
  namespace: tailscale
spec:
  interval: 30m
  chart:
    spec:
      chart: tailscale-operator
      version: '1.96.5'
      interval: 30m
      sourceRef:
        kind: HelmRepository
        name: tailscale
        namespace: tailscale
  values:
    operatorConfig:
      hostname: "homelab-tailscale-operator"
    apiServerProxyConfig:
      mode: "true"
```

Add the helm chart and release values.

However, the biggest piece to the puzzle are the secrets:

``` YAML
#This file MUST have name: operator-oauth
apiVersion: v1
kind: Secret
metadata:
    name: operator-oauth
    namespace: tailscale
type: Opaque
stringData:
    client_id: 0123456789abcdefghijklmnop
    client_secret: 0123456789abcdefghijklmnop
```

**You HAVE to encrypt with SOPS or another manager before committing to git**

And finally:

``` YAML
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: tailscale-namespace
  namespace: flux-system
spec:
  interval: 1h
  path: ./path/to/namespace.yaml
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: operator-oauth
  namespace: flux-system
spec:
  interval: 1h
  dependsOn:
    - name: tailscale-namespace
  path: ./path/to/secrets
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  decryption:
    provider: sops
    secretRef:
      name: sops-gpg
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: tailscale-config
  namespace: flux-system
spec:
  interval: 1h
  dependsOn:
    - name: operator-oauth # THIS IS THE KEY
  path: ./path/to/configs
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
```

## Why does this help?

The tailscale kubernetes operator automitically uses a secret named 'operator-oauth' with the stringData matching 'client_id' and 'client_secret' \
*meaning* \
as long as the secret is created before the config file is applied, the secret automatically gets pulled into tailscale.
