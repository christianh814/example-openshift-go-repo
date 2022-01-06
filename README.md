# OpenShift GitOps Repo

This is an example on how I would structure a 1:1 (repo-to-single cluster)
git repo for OpenShift. This can be modified for other, non OLM enabled,
Kubernetes clusters by just changing a few files.

This example assumes (as I mentioned in the 1:1 part above) that it's a
single repo for a single cluster. However, this can be modified (quite
easily) for poly/mono repos or for multiple clusters. This is meant as
a good starting point and not what your final repo will look like.

This is based on Argo CD but the same principals can be applied to Flux.

# Structure

Below is an explanation on how this repo is laid out. You'll notice
that I use [Kustomize](https://kustomize.io/) heavily. I do this since I
follow the [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
principal when it comes to YAML files.

```shell
cluster1-lbghf/ # 1
├── bootstrap # 2
│   ├── base
│   │   ├── kustomization.yaml
│   │   └── openshift-gitops-sub.yaml
│   └── overlays
│       └── default
│           ├── kustomization.yaml
│           ├── openshift-gitops-argocd.yaml
│           └── openshift-gitops-rbac-policy.yaml
├── components # 3
│   ├── applicationsets
│   │   ├── core-components-appset.yaml
│   │   ├── kustomization.yaml
│   │   └── tenants-appset.yaml
│   └── argocdproj
│       ├── kustomization.yaml
│       └── test-project.yaml
├── core # 4
│   ├── container-security-operator
│   │   ├── container-security-operator-sub.yaml
│   │   └── kustomization.yaml
│   └── openshift-gitops
│       └── kustomization.yaml
└── tenants # 5
    ├── bgd-blue
    │   ├── bgd-deployment.yaml
    │   └── kustomization.yaml
    └── myapp
        ├── kustomization.yaml
        ├── myapp-deployment.yaml
        ├── myapp-ns.yaml
        ├── myapp-route.yaml
        └── myapp-service.yaml
```
|#|Directory Name|Description|
|---|----------------|-----------------|
| 1. |&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`cluster1-lbghf`&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| This is the cluster name. This name should be unique to the specific cluster you're targeting. For OpenShift, I like to use the output of `kubectl get infrastructure cluster -o jsonpath='{.status.infrastructureName}'`|
| 2. | `bootstrap` | This is where bootstrapping specifc configurations are stored. These are items that get the cluster/automation started. They are usually install manifests. `base` is where are the "common" YAML would live and `overlays` are configurations specific to the cluster. The `kustomization.yaml` file in `default` has `cluster1-lbghf/components/applicationsets/` and `cluster1-lbghf/components/argocdproj/` as a part of it's `bases` config.|
| 3. | `components` | This is where specific components for the GitOps Controller lives (in this case Argo CD). `applicationsets` is where all the ApplicationSets YAMLs live and `argocdproj` is where the ArgoAppProject YAMLs live. Other things that can live here are RBAC, Git repo, and other Argo CD specific configurations (each in their repsective directories).|
| 4. | `core` | This is where YAML for the core functionality of the cluster live. Here is where the Kubernetes administrator will put things that is necissary for the functionality of the cluster. Under `openshift-gitops` is where you are using Argo CD to manage itself. The `kustomization.yaml` file uses `cluster1-lbghf/bootstrap/overlays/default` in it's `bases` configuration. This `core` directory gets deployed as an applicationset which can be found under `cluster1-lbghf/components/applicationsets/core-components-appset.yaml`. To add a new "core functionality" worokoad, one needs to add a directory with some yaml in the `core` directory. See the `container-security-operator` directory as an example.|
| 5. | `tenants` | This is where the workloads for this cluster live. Similar to `core`, the `tenants` directory gets loaded as part of an ApplicationSet that is under `cluster1-lbghf/components/applicationsets/tenants-appset.yaml`. This is where Devlopers/Release Engineers do the work. They just need to commit a directory with some YAML and the applicationset takes care of creating the workload. Note that `bgd-blue/kustomization.yaml` file points to another Git repo. This is to show that you can host your YAML in one repo, or many repos.|
