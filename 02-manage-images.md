# Manage images

## Understand private registry security

Several aspects govern private registry security and can be categorised into the following:

### Registry Authentication and Authorization

* **Credentials**: Access to private registries is typically controlled through username and password credentials. Podman needs to be configured with these credentials to access and interact with the registry.

* **Credential Storage**: Podman can store registry credentials securely using the `podman login` command. This command stores encoded credentials in a file (`$XDG_RUNTIME_DIR/containers/auth.json` or `~/.docker/config.json`) on the user's machine.

* **Authorization**: This is largely configured at the container registry level, the associated user(s) will be grated granular access to perform certain tasks (ie upload image, delete image, etc).

## Interact with many different registries

When you pull, push, or tag images with Podman, you can specify which registry to interact with by including the registry's URL in the image's name. The general format is `registry-url/repository-name/image-name:tag`.

Example of pulling from a specific registry:

```bash
podman pull myregistry.com/myimage:latest
```

Example of pushing to a specific registry:

```bash
podman push myimage:latest myregistry.com/myimage:latest
```

For private registries, or registries that require authentication, use the podman login command. This command will prompt for a username and password and store the credentials securely.

```bash
podman login myregistry.com
```

### Using multiple registry configurations

You might want to configure Podman to use multiple registries for pulling images, searching, etc. This can be done by editing the Podman configuration file, typically found at `/etc/containers/registries.conf`.

Example config:

```bash
[[registry]]
location = "myregistry.com"
insecure = false
blocked = false

[[registry]]
location = "otherregistry.com"
insecure = false
blocked = false
```

## Understand and use image tags

Image tags serve a number of important purposes:

### Version control of Images

* **Identifying Versions**: Tags are used to identify different versions of the same container image. This allows for version control, where you can have multiple versions of an image, each tagged differently.

* **Semantic Versioning**: Tags often follow semantic versioning (e.g., `2.1`, `2.1.3`) to indicate the version of the software or application contained in the image.

## Environment Specificity

* **Differentiating Environments**: Tags can be used to differentiate between environments like `development`, `testing`, `staging`, and `production`. For example, an image could be tagged as `myapp:prod` for production and `myapp:dev` for development.

## Push and pull images from and to registries

Given the following `Dockerfile`

```dockerfile
FROM nginx:latest

VOLUME /site-data

WORKDIR /usr/src/app

EXPOSE 80
```

We can `build`, `tag` and `push` it like so:

```bash
podman build /home/david/Downloads/podman/ --tag quay.io/myimage:0.1
podman push quay.io/myimage:0.1

# If auth is required

podman push quay.io/myimage:0.1 --creds USERNAME:PASSWORD
```

To pull images:

```bash
podman pull nginx:latest
```

## Back up an image with its layers and meta data vs. backup a container state

A container is a running instance of an image, therefore approaches to back up differ between the two

To back up an image:

```bash
podman save --quiet -o myimage.tar imageID
podman save --format docker-dir -o ubuntu-dir ubuntu
podman save > alpine-all.tar alpine:latest
```

To restore:

```bash
podman load -i myimage.tar
```

In summary, backing up an image is like saving a recipe, while backing up a container's state is like preserving a meal that's already been cooked. Each serves different purposes and comes with its own set of considerations and methodologies.

Backing up a running container primarily involves backing up any attached volumes or using specific application level tooling (ie `mysqldump`).

If applicable, `podman volume export` can also be used.
