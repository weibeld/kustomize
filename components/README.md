TODO:

Create base with Deployment, Namespace, ServiceAccount, Role, RoleBinding, PodSecurityPolicy.

Possible combinations of features when using this base:

- Deployment
- Deployment, Namespace
- Deployment, ServiceAccount
- Deployment, ServiceAccount, Role, RoleBinding, PodSecurityPolicy
- Deployment, Namespace, ServiceAccount, Role, RoleBinding, PodSecurityPolicy

Can't be easily done if the base hardcodes all features.

Refactor with kustomize components.

Create base containing only the Deployment.

Create three components:

- Namespace
- ServiceAccont
- PodSecurityPolicy (check if it can reference the ServiceAccount component)

Create different overlays by freely combining the available components.

* * * 

Why not use normal overlays instead of components?

> The simplest way to implement this in Kustomize is to create an overlay for each component, and have a top-level overlay include only the components that it requires. The problem with this approach is that the mechanism that Kustomize uses to interpret and ultimately apply patches does not allow for mutating different aspects of the very same base resource in parallel without modifying its GVKN (unique identifier). That is, sibling overlays that operate on the same level of the hierarchy and are not chained (i.e. one does not import the other as base) cannot modify their common parent due to resource ID conflicts.

https://github.com/arrikto/kubernetes-enhancements/blob/master/keps/sig-cli/1802-kustomize-components.md#motivation

Overlays cannot be chained, i.e. a given overlay cannot build upon what an earlier overlay did (or two overlays cannot modify their common base).
