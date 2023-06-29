## Kubernetes GitOps Configuration Example with Argo CD  [![Twitter](https://img.shields.io/twitter/follow/piotr_minkowski.svg?style=social&logo=twitter&label=Follow%20Me)](https://twitter.com/piotr_minkowski)

Here's the current structure of the repo (will be modified soon):
```shell
├── apps
│   └── cert-manager
│       ├── secrets.yaml
│       ├── certificates.yaml
│       └── cluster-issuer.yaml
└── bootstrap
    ├── Chart.yaml
    ├── templates
    │   ├── cert-manager.yaml
    │   └── projects.yaml
    └── values.yaml
```

For now, it shows how to install and use in the GitOps way:
1. [cert-manager](https://cert-manager.io/) via Helm chart

For more information you can refer to the articles:
1. [Testing GitOps on Virtual Kubernetes Clusters with ArgoCD](https://piotrminkowski.com/2023/06/29/testing-gitops-on-virtual-kubernetes-clusters-with-argocd/)

## Getting Started

You can test the configuration on the virtual Kubernetes cluster using the [vcluster](https://www.vcluster.com/) tool. In order to that create the ArgoCD `ApplicationSet` based on the cluster generator:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: config
  namespace: argocd
spec:
  generators:
    - clusters: # (1)
        selector:
          matchLabels:
            argocd.argoproj.io/secret-type: cluster
  template:
    metadata:
      name: '{{name}}-config-test'
    spec:
      destination:
        namespace: argocd
        server: https://kubernetes.default.svc
      project: default
      source:
        helm:
          parameters:
            - name: server
              value: '{{server}}'
            - name: project
              value: '{{metadata.labels.loft.sh/vcluster-instance-name}}'
        path: bootstrap
        repoURL: https://github.com/piomin/kubernetes-config-argocd.git
        targetRevision: '{{metadata.labels.loft.sh/vcluster-instance-name}}'
      syncPolicy:
        automated: {}
        syncOptions:
          - CreateNamespace=true
```

If you just want to test on the single cluster managed by ArgoCD you can create the following ArgoCD `Application`:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: config
  namespace: argocd
spec:
  destination:
    namespace: argocd
    server: https://kubernetes.default.svc
  project: default
  source:
    path: bootstrap
    repoURL: https://github.com/piomin/kubernetes-config-argocd.git
    targetRevision: HEAD
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
```