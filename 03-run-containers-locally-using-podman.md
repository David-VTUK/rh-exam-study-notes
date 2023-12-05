# Run containers locally using Podman

## Get container logs

`podman logs $ContainerID` can be ued to retrieve container logs.

Can be redirected to an output - ie `podman logs ebe > logs.txt`

## Listen to container events on the container host

`podman events` can be used to accomplish this.

Examples:

```bash
#Get all events from all containers
podman events
```

```bash
#Get events from a specific container
podman events --filter container=my-container
```

```bash
#Get events by event type (e.g., create, start, stop, remove):
podman events --filter event=start
```

```bash
#Get events by time range
podman events --since 10m --until 5m
```

## Use Podman inspect

`podman inspect` can be used against multiple object types, ie:

`podman inspect [options] {CONTAINER|IMAGE|POD|NETWORK|VOLUME} [...]`

For example, to inspect an image:

```bash
podman inspect a6bd71f48f68
```

## Specifying environment parameters

At runtime, `-e or --env` can be used to specify an environment variable.

```bash
podman run -e "MYVAR=myvalue" <image>
```

For multiple environment variables, repear the `-e` flag:

```bash
podman run -e "VAR1=value1" -e "VAR2=value2" <image>
```

Environment variables can also be loaded from a file:

```bash
podman run --env-file myenvfile.env <image>
```

`-e` can also be used to overwrite an environment variable defined in the container image.

## Expose public applications

## Get application logs

## Inspect running applications