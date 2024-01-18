# Deploy Applications

## Deploy applications from resource manifests

```bash
# Apply the configuration in pod.json to a pod
oc apply -f ./pod.json

# Apply resources from a directory containing kustomization.yaml - e.g. dir/kustomization.yaml
oc apply -k dir/

# Apply the JSON passed into stdin to a pod
cat pod.json | oc apply -f -

# Apply the configuration from all files that end with '.json' - i.e. expand wildcard characters in file names
oc apply -f '*.json'
```

## Use Kustomize overlays to modify application configurations

Kustomize is a templating tool for Kubernetes manifests in its native form (Yaml). When working with raw YAML files you will typically have a directory containing several files identifying the resources it creates. To begin, a directory containing our manifests needs to exist:

```shell
/home/david/app/base
total 16
drwxrwxr-x  2 david david 4096 Feb  9 11:44 .
drwxr-xr-x 27 david david 4096 Feb  9 11:44 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

This will form our `base` - we will build on this but adding customisations in the form of overlays. First, we need a `kustomize` file. which can be created with `kustomize create --autodetect`

This will create kustomization.yaml in the current directory:

```shell
total 20
drwxrwxr-x  2 david david 4096 Feb  9 11:47 .
drwxr-xr-x 27 david david 4096 Feb  9 11:47 ..
-rw-rw-r--  1 david david  340 Feb  9 11:09 deployment.yaml
-rw-rw-r--  1 david david  108 Feb  9 11:47 kustomization.yaml
-rw-rw-r--  1 david david  153 Feb  9 11:09 service.yaml
```

The contents being:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
```

### Variants and Overlays

* variant - Divergence in configuration from the `base`
* overlay - Composes variants together 

Say, for example, we wanted to generate manifests for different environments (prod and dev) that are based from this config, but have additional customisations. In this example we will create a `dev` variant encapsulated in a single Overlay

```shell
mkdir -p overlays/{dev,prod}
cd overlays/dev 
```
Begin by creating a Kustomization object specifying the base (this will create `kustomization.yaml`) :

```shell
kustomize create --resources ../../base
```
In this example, I want to change the replica count to 1, as it's a dev environment. In the `dev` directory, create a new file `deployment.yaml` containing:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
```

The `kustomization.yaml` file needs modifying to include a `patchesStrategicMerge` block:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
  - ../../base
patchesStrategicMerge:
  - deployment.yaml
```
Patches can be used to apply different customizations to Resources. Kustomize supports different patching mechanisms through `patchesStrategicMerge` and `patchesJson6902`. `patchesStrategicMerge` is a list of file paths.

We can generate the manifests and apply to the cluster by executing (from the base folder):

```shell
kustomize build ./overlay/dev | kubectl apply -f -
```

By running this, only 1 pod will be created in the deployment object, instead of what's defined in the `base` because of the customisation we've applied. We can do the same with prod, or any arbitrary number of environments.

## Deploy applications from images, OpenShift templates, and Helm charts

### Images

Aside from referencing specific images in manifests, the `oc new-app` command can also be used:

```bash
oc new-app --help

This command will try to build up the components of an application using **images**, templates, or code that has a public
repository. It will look up the images on the local container storage (if available), a container image registry, an
integrated image stream, or stored templates

# Deploy from a public registry image
oc new-app <docker-registry-url>/<image-name>:<tag>

# Deploy from an image stream in the OCP cluster
oc new-app --image-stream=<image-stream-name>:<tag>

# Deploy from a internal registry
oc new-app <internal-registry-url>/<namespace>/<image-name>:<tag>
```

## Deploy jobs to perform one-time tasks

```bash
oc create job NAME --image=image [--from=cronjob/name] -- [COMMAND] [args...] [options]
```



## Manage application deployments

## Work with replica sets

## Work with labels and selectors

## Configure services

## Expose both HTTP and non-HTTP applications to external access

## Work with operators such as MetalLB and Multus
