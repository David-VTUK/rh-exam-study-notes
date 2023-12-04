# Implement images using Podman

## Understand and use FROM (the concept of a base image) instruction.

`FROM` serves as a starting point for building a container image. It specifies the `base image` to use for the image. This base image is typically a lightweight version of an operating system like Ubuntu, Alpine, or even a minimal image that contains only the bare essentials needed for an application to run.

Example:

```dockerfile
FROM alpine:3.18.5
```

Specifies the image will be using the `alpine:3.18.5` image as its base. Represented as `name:version(aka tag)`

Other key characteristics of `FROM` include:

**Layering** - Every instruction in a Dockerfile will result in a `layer` when the image is built. This is partly why we specify a base image first - it provides us with a foundation to add further components to the image. 

**Multi Stage Container** - You can have multiple `FROM` declarations in a container image - these are used in multi-staged builds. This is useful for creating lightweight images by compiling or building your application in a first stage with all necessary build tools, and then copying the relevant artifacts into a second stage with a leaner base image.

**Efficiency and Security** - Selecting (or building) an appropriate base image is crucial for the efficiency and security of the container image. A minimal base image reduces the size of the image and also minimizes the attack service, as there are fewer components that could contain vulnerabilities compared to a fully fledged, general purpose Operating System.

Example Multi Stage Build:

```dockerfile
# Stage 1: Building the code
FROM node:14 as builder

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
COPY package*.json ./
RUN npm install

# Bundle app source
COPY . .

# Build the application
RUN npm run build

# Stage 2: Setup the production environment
FROM node:14-alpine

# Set the working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install only production dependencies
RUN npm install --only=production

# Copy built assets from the builder stage
COPY --from=builder /usr/src/app/dist ./dist

# Your app runs on port 3000
EXPOSE 3000

# Start the app
CMD ["node", "dist/main.js"]

```

* Stage 1: Building code  
  * `FROM node:14 as builder`: this stage starts with a  Nodejs version 14 image. `as builder` part names this stage as `builder`.
  * The image then copies the application's source code and its dependencies into the image and performs the build. In this case, it's a typical Node.js setup with npm install and a build command like npm run build.
* Stage 2: Setting up runtime environment
  * Note this stage is using a slightly different container image, although named similarly. In this example `FROM node:14-alpine` - a more lightweight version of the node image from stage 1.
  * This stage does not include the entire build environment. Instead, it only installs the production dependencies (`npm install --only=production`) and copies the built assets (dist directory) from the builder stage. This way, the final image contains only what's necessary to run the application, reducing the image size and surface area for potential vulnerabilities.


## Understand and use RUN instruction.  

`RUN` is a fundamental command used during the image building process. It allows us to execute commands in a layer above the current image and commit the results. The resulting committed layer is used in the subsequent steps in the Image definition.

`RUN` is essential for installing software, building software, altering files and running tasks necessary to set up the desired environment inside the container for our application

Example:

```dockerfile
FROM fedora:39

RUN dnf upgrade --refresh && dnf install -y $packageName
```

Key characteristics of `RUN`:

* Layering
  * Each `RUN` command creates a new layer inside the container image. For efficiency, it is common to combine multiple `RUN` commands together using `&&`, as shown in the example above.
* Shell vs exec form
  * The `RUN` command can be run in two different ways - `shell` and `exec`
    * Shell method: `RUN <command>` - is the equivalent of invoking `/bin/sh -c` followed by the `<command>`
    * Exec method: `RUN ["executable", "param1", "param2"]` - This form does not invoke a shell. It's useful when you want to avoid shell string munging, and it's necessary if you want to execute software that's not a Unix shell, like Java or Python programs.

Example:

```dockerfile
# Using the shell form to update package lists and install a package (shell method)
RUN dnf upgrade --refresh && dnf install -y curl
```

```dockerfile
# Invoking a python app (exec method)
RUN ["python", "/path/to/your_script.py", "arg1", "arg2"]

```

## Understand and use ADD instruction.  

`ADD` is used to copy files, directories from either a local machine building the image, or from a remote URL. `ADD` can be considered to have all the functionality of `COPY` but with more functionality.

Key difference: `COPY` does **not** support remote URL's. 

Key characteristics of `ADD`

* **Automatic Tar Extraction** - If the source file being copied is a tarball in recognized compression formats (like gzip, bzip2, etc.), ADD automatically extracts the contents of the tarball into the destination directory within the container’s filesystem. This is not the case with COPY, which would just copy the tar file as-is.

* **Syntax** - `ADD <src> <dest>` : Here, `<src>` could be a file or directory from the Dockerfile's context, or a URL. `<dest>` is the path where the files will be placed inside the Docker image.

Examples:

```dockerfile
# Copies a local file "example.txt" into the image at "/path/in/container/"
ADD example.txt /path/in/container/
```

```dockerfile
# Copies a local directory "config_files" and its contents into "/config/" inside the image
ADD config_files /config/
```

```dockerfile
# Downloads a file from a remote URL and places it in "/path/in/container/"
ADD https://example.com/download/somefile /path/in/container/
```

```dockerfile
# Automatically extracts "app.tar.gz" into "/app/" directory inside the image
ADD app.tar.gz /app/
```


**Considerations and Best Practices**

* Use `COPY` When Possible: Because of its additional features, ADD can lead to unexpected behavior (like automatic tar extraction) which might not be desired in all cases. Therefore, it's generally recommended to use the simpler `COPY` command when you just need to copy files from the local context without the extra features of `ADD`.

* Avoid Using `ADD` for Remote URLs: While `ADD` can fetch files from remote URLs, this is generally discouraged. It's better to use a tool like `curl` or `wget` in a RUN command to fetch remote files, which gives you more control over the download process (such as error handling and retries).

* Image Size Considerations: Be mindful of the files you add to your image. Adding unnecessary files or large files can increase the size of your image significantly.

* The choice between ADD and COPY depends on your specific needs. For most file-copying tasks within the Dockerfile's context, COPY is sufficient and preferable. Use ADD for its unique features when they are explicitly needed.

## Understand and use COPY instruction.  

The `COPY` command in a Dockerfile is used to copy files and directories from the host file system into the Docker image. It is a straightforward and commonly used command in Dockerfile scripting, particularly for adding local files to the image.

`Copy` takes two parameters - a source and destination and can be used to copy individual files as well as entire directories. 

* The source path must be inside the context of the build (the directory where the Dockerfile is located or specified via the build context).  

* The destination path is inside the container's filesystem. It can be an absolute path or a path relative to `WORKDIR` if it has been set.

Example:

```dockerfile
FROM fedora:39

RUN dnf upgrade --refresh && dnf install -y $packageName

# Copy the contents of the folder `app` in the working directory of the dockerfile to /usr/src/app inside the container

COPY ./app /usr/src/app
```

## Understand the difference between ADD and COPY instructions.  

The `ADD` and `COPY` commands in Dockerfiles are both used to copy files from the build context into the Docker image. However, they have distinct functionalities and are suitable for different scenarios. Here are the main differences:
COPY Command

* Basic Functionality: COPY is a straightforward command used to copy files and directories from the Dockerfile's context (the local file system) into the Docker image.
* Simplicity and Clarity: It is explicitly used for copying local files and does not have any additional functionality, making it clear and predictable.
* Best Practice: Docker's best practices recommend using COPY unless you specifically need the additional capabilities of ADD.

**ADD Command**

* Advanced Features: ADD has all the capabilities of COPY but also supports a couple of additional features:
    * **Remote URL Support**: ADD can copy files from a remote URL (e.g., http:// or https://) into the image, which COPY cannot do.
    * **Automatic Tar Extraction**: ADD can automatically extract a tarball from the source directly into the destination within the image's filesystem. This extraction happens only if the tar file is in a recognized compression format (like gzip, bzip2, etc.) and is a local tar file (not a URL).

* **Versatility but with Complexity**: Because of its additional capabilities, ADD can be used for more complex tasks than COPY. However, this also means its behavior can be less predictable, especially for new users.

* **Use with Caution**: Given its additional features, ADD can potentially introduce unwanted side-effects (like unexpected network downloads or automatic extraction of files), and hence should be used carefully.

**Choosing Between COPY and ADD**

* Use `COPY` when you simply need to copy files from your local context into the Docker image. It is straightforward and adheres to the principle of least surprise.
* Use `ADD` when you need to utilize its advanced features like fetching files from a URL or automatic tarball extraction. However, be cautious of its implications, and prefer `COPY` for most other scenarios.

## Understand and use WORKDIR and USER instructions.  

`WORKDIR` is used to set the working directory for subsequent commands such as `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD`.

Setting the `WORKDIR` provides context for these commands, as this provides the location it is based from.

Examples:

```dockerfile
FROM fedora:39

RUN dnf upgrade --refresh && dnf install -y $packageName

WORKDIR /usr/

RUN echo "HI" >  test.txt
```

The file created will reside as /usr/test.txt

```dockerfile
FROM fedora:39

COPY ./app /usr/src/app

WORKDIR /usr/src/app

RUN ["python", "your_script.py", "arg1", "arg2"] 
```
This will run `your_script.py` without having to specify the absolute path

`USER` is used to set the user identity (UID) and optionally the user group (GID) for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions.

It's commonly used to define a specific user to run an application inside the container vs running it as `root`, which can hae security implications.

Example:

```dockerfile
# Create a user 'appuser' and set it as the current user
RUN adduser --disabled-password --gecos '' appuser
USER appuser
```

**Syntax**

`USER <user>` Sets the user for subsequent commands. `<user>` can be a user name or UID.

`USER <user>:<group>` Sets both the user and group for subsequent commands. `<group>` can be a group name or GID.

## Understand security-related topics.  

??

## Understand the differences and applicability of CMD vs. ENTRYPOINT instructions.  

`CMD` and `ENTRYPOINT` are two commands in Dockerfile that are often used to define the default behavior of a Docker container when it starts. While they appear similar, they have different purposes and behaviors.

**`CMD`**

* **Purpose**: CMD specifies the default command to run when a container starts. It is intended to provide defaults for an executing container.  
* **Flexibility**: If you run the container and pass in arguments, those arguments will replace the default specified in the CMD.
* **Syntax Variations:**
    * Shell form: `CMD command param1 param2`
    * Exec form: `CMD ["command", "param1", "param2"]`
* **Use Case**: Use CMD when you need to provide a default command and/or parameters that users can easily override when starting a container.

**`ENTRYPOINT`**

* **Purpose**: `ENTRYPOINT` configures a container to run as an executable. It allows you to set the base command which gets executed every time the container starts.
* **Consistency**: Unlike `CMD`, if you pass arguments to the container at runtime, they are appended to the ENTRYPOINT command.
* **Syntax Variations**:
    * Shell form: `ENTRYPOINT command param1 param2`
    * Exec form (preferred for signal handling): `ENTRYPOINT ["command", "param1", "param2"]`
* **Use Case**: Use ENTRYPOINT when you want to configure your container to run as a specific executable (like a web server, database, etc.) and you don’t want the base command to be easily overridden.

You can also combine the two:

```dockerfile
ENTRYPOINT ["executable"]
CMD ["param1", "param2"]
```

Here, `executable` will always run when the container starts, and it will use `param1` and `param2` as default parameters unless overridden.

## Understand ENTRYPOINT instruction with param.

* **Syntax Variations**:
    * Shell form: `ENTRYPOINT command param1 param2`
    * Exec form (preferred for signal handling): `ENTRYPOINT ["command", "param1", "param2"]`


## Understand when and how to expose ports from a Containerfile.  

`EXPOSE` is used to define network services offered by the resulting container. When a container runs applications that listen on specific ports for incoming connections, we define these with `EXPOSE` so external applications/clients can access these services.

Unless explicitly defined, TCP is assumed. 

Examples:

```dockerfile
# Listen on port 80 (TCP)
EXPOSE 80
```

```dockerfile
# Listen on ports 80 and 443 (TCP)
EXPOSE 80 443
```
```dockerfile
# Listen on port 8000 (UDP)
EXPOSE 8000/udp
```

## Understand and use environment variables inside images.  


## Understand ENV instruction.  


## Understand container volume.  


## Mount a host directory as a data volume.  


## Understand security and permissions requirements related to this approach.  


## Understand the lifecycle and cleanup requirements of this approach. 