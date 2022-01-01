# Image Transformer Customisation

This shows how to customise the image transformer for a custom resource.

## Background

Kustomize includes various [transformers](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/transformerconfigs/README.md) that allow to modify certain fields of the generated resources by setting certain fields in the kustomization file.

For example, the namespace transformer allows to set the following field in the kustomization file:

```yaml
namespace: foo
```

And the namespace transformer will set the namespace of every processed resource to `foo`.

Similarly, the [image transformer](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/transformerconfigs/README.md#images-transformer) allows to set the following field:

```yaml
images:
  - name: postgres
    newName: my-registry/my-postgres
    newTag: v1
  - name: nginx
    newTag: 1.8.0
```

And the image transformer will set the names and tags of the listed container images in the applicable processed resources accordingly.

## Customising the image transformer

The image transformer natively looks for `image` fields in the processes resources one level below any `containers` or `initContainer` fields (see [documentation](https://github.com/kubernetes-sigs/kustomize/tree/master/examples/transformerconfigs#images-transformer)). This is where the images are defined in the native Kubernete resources, such as Pods, Deployments, DaemonSets, etc.

But what if you want to apply the image transformer to a custom resource that includes a container image specified at a different location?

In this case, the image transformer must be indicated where there's an image definition in this custo resource.

This is described in [_Images transformations_](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/transformerconfigs/images/README.md) in the kustomize documentation.

## Example

Consider the following custom resource in `resources.yaml`:

```yaml
apiVersion: config/v1
kind: MyKind
metadata:
  name: test
spec:
  containers:
    - image: nginx:latest
      name: nginx
  runLatest:
    container:
      image: alpine:latest
      name: alpine
```

It defines two container images at two different locations: first, under the `containers` field, and, second, under a `runLatest.container` field.

Now, consider the following kustomization file:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - resources.yaml

images:
  - name: nginx
    newName: myregistry.io/nginx
    newTag: 1.19.7
  - name: alpine
    newName: myregistry.io/alpine
    newTag: 3.13.2
```

It attempts to update the image specifications by prepending a private container registry identifier to each name and also set the image tags to a deterministic version.

If you run `kustomize build` with this configuration, the output will be: 

```yaml
apiVersion: config/v1
kind: MyKind
metadata:
  name: test
spec:
  containers:
  - image: myregistry.io/nginx:1.19.7
    name: nginx
  runLatest:
    container:
      image: alpine:latest
      name: alpine
```

The first container image (`nginx`) has been correctly updated, but the second one (`alpine`) hasn't been changed.

This is because the first image specification in the custom resource is located under a `containers` field. This is the default location that the image transformer natively looks for `image` fields.

However, the second image specification in the custom resource is located at a custom path (`spec.runLatest.container`). This location isn't known by the image transformer, thus it didn't update this image specification.

To fix this, save the following in a file named `images-config.yaml`:

```yaml
images:
  - kind: MyKind
    path: spec/runLatest/container/image
```

The above is a configuration for the image transformer. It tells the image transformer that the custom resource type `MyKind` has an image specification at the path `spec/runLatest/container/image`.

Now, reference this configuration from your `kustomization.yaml` file as follows:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - resources.yaml

images:
  - name: nginx
    newName: myregistry.io/nginx
    newTag: 1.19.7
  - name: alpine
    newName: myregistry.io/alpine
    newTag: 3.13.2

configurations:
  - images-config.yaml
```

Now, rerun `kustomize build`, and the output should be:

```yaml
apiVersion: config/v1
kind: MyKind
metadata:
  name: test
spec:
  containers:
  - image: myregistry.io/nginx:1.19.7
    name: nginx
  runLatest:
    container:
      image: myregistry.io/alpine:3.13.2
      name: alpine
```

The second container image at the custom location has now been also correctly updated as defined in your kustomization file.
