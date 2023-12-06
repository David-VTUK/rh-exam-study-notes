# Troubleshoot containerized applications

## Understand the description of application resources

Understanding the description of application resources involves leveraging a number of `podman` commands.

Basic container inspection can be done with `podman inspect [container_name_or_id]` from this we can get

* ID
* Args
* Image
* Pod
* Volume
* Network

## Get application logs

`podman logs container_name_or_id]` *if* the container writes to `stdout` / `stderr`. Otherwise it may log to a file or directly to an external logging aggregator

## Inspect running applications

`podman inspect` can also be used to accomplish this. IE to identify any exposed ports.

Health checks can be run by executing `podman healthcheck run [container_name_or_id]`. Note, this requires health checks to be configured on the container. An example of which can be defined in the compose file:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 2m
```

Changes to the containers filesystem can be identified with `podman diff [container_name_or_id]`

## Connecting to running containers

`podman exec -it [container_name_or_id] /bin/sh` (change `sh` to an appropriate shell). This will create a interactive shell directly inside the container subsequent commands can be run.

If we know exactly what we need to run in the container and log the output to `stdout`, we can use

```bash
podman exec [container_name_or_id] [command]
```

For example:

```bash
podman exec [container_name_or_id] ls /path/to/directory
```

We can also specify a specific user to run the command as, using the `user` flag:

```bash
podman exec --user [username] [container_name_or_id] [command]
```
