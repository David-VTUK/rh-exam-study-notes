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

Exposing applications using Podman involves mapping ports from the container to the host system. This way, we can access services running inside the container from outside the host machine. There's a number of ways to achieve this:

```bash
#Expose a single port from the host system and map it to the container. In this example, map port 5000 from the host to port 80 in the container
#Format is -p <host-port><container-port>

podman run -d -p 5000:80 nginx
curl localhost:5000
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

To expose multiple ports, follow the same pattern for specifying multiple environment variables, ie multiple `-p` flags:

```bash
podman run -d -p 5000:80 -p 6000:443 nginx
```

TCP is assumed as the port type, to specify UDP:

```bash
podman run -p 5000:5000/udp <image>
```

To expose a range of ports:

```bash
podman run -p 8000-8005:8000-8005 <image>
```

In systems with multiple interfaces, we can specify one the hosts IP addresses in the binding:

```bash
podman run -p 172.16.10.45:80:80 nginx
```

## Get application logs

Container logs (managed by Podman) give you a view of what is happening at the container level, application logs (managed by the application itself) provide insights into the behavior of the application inside the container.

With Container logs, these can be accessed by `podman logs [container_name_or_id]`. This will capture whatever the main process writes to stdout and stderr.

Application logs may do the same, or write to a file / access a logging aggregator directly. This is dependent on the configuration of the containers application.

For example, if a container writes to `/var/log/app.log`, these won't be shown in `podman logs`. To retrieve this, we'd need to either mount the directory to the host, or exec into the container to view:

```bash
podman exec -it [container_name_or_id] bash
cat /var/log/app.log
```

Or as a one-liner

```bash
podman exec -it [container_name_or_id] cat var/log/dpkg.log
```

## Inspect running applications

There are a number of commands we can use to inspect a running application/

### Container details

`podman inspect [container_name_or_id]` can be used to get information about a container, including its configuration, state, network sections and more.

## Container logs

`podman logs [container_name_or_id]` can be used to get everything that has been written to `stderr` and `stdout`.

### Container processes

`podman top [container_name_or_id]` can be used to view the running processes inside a container.

### Container resources

`podman stats` can be used to view resource usage of containers, such as CPU, memory, network IO, Disk IO, etc.

### Container shell

`podman exec -it [container_name_or_id] /bin/bash` can be used *if* the container has been built with a shell installed (not all will). Once a shell has been opened, you can inspect the container.

### Container health check

`podman healthcheck run [container_name_or_id]` can be used to manually trigger a health check, `podman inspect [container_name_or_id]` can be used to view the results after.

### Container filesystem changes

`podman diff [container_name_or_id]` can be used to identify what changes have been made by the running application to the local filesystem.
