This shows that multiple (sibling) overlays cannot modify the same base
resource (i.e. a resource with the same GVKN identifier). Attempting to do so
results in the error "may not add resource with an already registered id:
apps_v1_Deployment|~X|example". This is because overlays clone and modify base
resources in parallel.

This is the main motivation for kustomize components, which can do this.

This is described in the [kustomize components design doc](https://github.com/arrikto/kubernetes-enhancements/blob/master/keps/sig-cli/1802-kustomize-components.md#motivation):

> The simplest way to implement this in Kustomize is to create an overlay for each component, and have a top-level overlay include only the components that it requires. The problem with this approach is that the mechanism that Kustomize uses to interpret and ultimately apply patches does not allow for mutating different aspects of the very same base resource in parallel without modifying its GVKN (unique identifier). That is, sibling overlays that operate on the same level of the hierarchy and are not chained (i.e. one does not import the other as base) cannot modify their common parent due to resource ID conflicts.

This issue is also reported in [kustomize/issues/1251](https://github.com/kubernetes-sigs/kustomize/issues/1251).
