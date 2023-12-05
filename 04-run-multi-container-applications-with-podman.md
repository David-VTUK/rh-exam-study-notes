# Run multi-container applications with Podman

## Create application stacks

Broadly speaking, application stacks involve the use of multiple containers that work together to form a complete application system. This can be accomplished in two different ways - Podman Pods and Podman-compose, as detailed below.

### Using Podman Pods

A podman `pod` is a group of one or more containers that share the same network namespace, amongst other resources.

To create a Podman pod:

```bash
podman pod create --name mypod -p 8080:80
```

Note: we don't have anything running this pod, it's a shell.  We'll add a `nginx` container inside this pod:

```bash
podman run -d --pod mypod nginx
```

Which we can then access by `curl localhost:8080`

To add a second container to this pod:

`podman run -d --pod mypod redis`

Which we can inspect with

```bash
podman pod ps

POD ID        NAME        STATUS      CREATED        INFRA ID      # OF CONTAINERS
6babca889ba2  mypod       Running     5 minutes ago  a45cc929c682  3
```

Note: it states 3 containers because of the `pause` container.  

### Using Podman-compose

## Understand container dependencies

## Working with environment variables

## Working with secrets

## Working with volumes

## Working with configuration
