# Docker CLI Notes (New Command Style)

Simple, topic-wise explanation of every command. Format: `docker <module> <action>`, for example `docker container run`.

---

## 1. Getting Started

| Command | What it does |
|---|---|
| `docker --help` | Lists all modules (parent commands) like `container`, `image`, `network`, `volume`. |
| `docker container --help` | Lists sub-commands (child commands) of a module. |
| `docker network ls` | Shows all Docker networks. |
| `docker images --all` | Shows all images stored on your machine (local repo). |

---

## 2. Running Containers

| Command | What it does |
|---|---|
| `docker container run ubuntu` | Creates and starts a container from the `ubuntu` image. If the image is not on your machine, Docker downloads it from Docker Hub first. |
| `docker container run ubuntu <command>` | Runs a specific command inside the new container. |
| `docker container run fedora sleep 30` | Container runs for 30 seconds, then stops. |
| `docker container run -d ubuntu sleep 30` | `-d` (detach) runs it in the background so your terminal is not blocked. |
| `docker container run -it ubuntu /bin/bash` | `-it` (interactive + terminal) gives you a shell inside the container. |
| `docker container run -d -p 7770:80 --name web nginx` | Runs nginx in background, names it `web`, and maps host port 7770 to container port 80. |

**Leaving an interactive container**
- `exit` or `Ctrl+D`: leaves the container and **stops** it.
- `Ctrl+P` then `Ctrl+Q`: leaves the container but keeps it **running** in the background.

---

## 3. Listing Containers

| Command | What it does |
|---|---|
| `docker container ls` | Shows only running containers. |
| `docker container ls -a` | Shows all containers (running and stopped). |
| `docker container ls -aq` | Shows only the IDs of all containers (`-q` = quiet). |

---

## 4. Container Lifecycle (Start, Stop, Remove)

| Command | What it does |
|---|---|
| `docker container start <id>` | Starts a stopped container. |
| `docker container stop <id>` | Stops a running container gracefully (gives it time to shut down). |
| `docker container kill <id>` | Stops a container forcefully and immediately. |
| `docker container restart <id>` | Stops and starts the container again. |
| `docker container pause <id>` | Freezes all processes inside the container. |
| `docker container unpause <id>` | Resumes a paused container. |
| `docker container wait <id>` | Waits until the container stops, then prints its exit code. |
| `docker container rename <id> <new-name>` | Changes the container's name. |
| `docker container rm <id>` | Removes a stopped container. |
| `docker container rm $(docker container ls -aq)` | Removes all stopped containers at once. Running ones must be stopped first. |
| `docker container prune` | Deletes all stopped containers (asks for confirmation). |

---

## 4.5 Working Inside a Running Container

| Command | What it does |
|---|---|
| `docker container exec -it <id> /bin/bash` | Opens a new shell inside a running container. Exiting does **not** stop the container. |
| `docker container attach <id>` | Brings a background container to the foreground (shows its live output). |

---

## 5. Inspecting and Monitoring

| Command | What it does |
|---|---|
| `docker container inspect <id>` | Shows full details and configuration (IP, mounts, env, etc.) in JSON. |
| `docker container logs <id>` | Shows the container's output logs (for example nginx access logs). |
| `docker container top <id>` | Shows processes running inside the container. |
| `docker container stats` | Live CPU, memory, and network usage of containers. |
| `docker container port <id>` | Shows the port mapping of the container. |
| `ps -aux` | Linux command: shows all processes on your machine. |
| `netstat -nltp` | Linux command: shows open (listening) ports on your machine. |

---

## 6. Port Forwarding

By default a container is only reachable inside the Docker host. To reach it from another machine, forward a host port to a container port.

```bash
docker container run -d -p 7770:80 --name web nginx
```

Meaning: any request coming to the host on port **7770** is sent to the container's port **80**.

Open in a browser: `http://<public-ip-of-ec2>:7770`

(On EC2, make sure port 7770 is allowed in the Security Group.)

---

## 7. Create, Diff and Copy

| Command | What it does |
|---|---|
| `docker container create ubuntu sleep 30` | Creates a container (downloads image if needed) but does **not** start it. The `sleep 30` runs only after you `start` it. |
| `docker container diff <id>` | Shows file changes made inside the container since it was created. |
| `watch 'docker container diff <id>'` | Re-runs the diff every 2 seconds so you can watch changes live. |
| `docker container cp <local-file> <id>:/path` | Copies a file from host into the container. |
| `docker container cp <id>:/path <local-path>` | Copies a file from container to host. |

Diff letters: **A** = Added, **C** = Changed, **D** = Deleted.

---

## 8. Export and Import (Container to Image)

| Command | What it does |
|---|---|
| `docker container export <id> -o pk.tar` | Saves the container's filesystem as a tar file. |
| `ls -lh` | Linux command: shows file sizes (to check the export file). |
| `docker image import pk.tar <image-name>` | Creates a new image from that tar file. |
| `docker image ls` | Lists images (check that the new one is there). |

---

## 9. Create an Image from a Running Container (Commit)

```bash
docker container commit --author "yourname" -m "your message" <container-id> <new-image-name>
```

Takes a snapshot of the container (with all your changes) and saves it as a new image.

---

## 10. Docker Hub (Public Registry)

| Command | What it does |
|---|---|
| `docker login` | Logs in to Docker Hub with username and password. |
| `docker pull ubuntu` | Downloads the `ubuntu` image (latest tag). |
| `docker pull ubuntu:<tag>` | Downloads a specific version, for example `ubuntu:20.04`. |
| `docker image tag <image> <dockerhub-user>/<repo>:<tag>` | Gives the image a name that matches your Docker Hub repo. |
| `docker push <dockerhub-user>/<repo>:<tag>` | Uploads the image to your Docker Hub repo. |
| `docker pull hackerboypk/hackerbos:latest` | Downloads your image from Docker Hub (syntax shown on the Hub page). |

---

## 11. Image Commands

| Command | What it does |
|---|---|
| `docker image ls` (same as `docker images`) | Lists local images. |
| `docker image ls --format '{{.ID}},{{.Repository}}'` | Lists images in a custom format. More options in the docs. |
| `docker image history <image>` | Shows the layers and commands used to build the image. |
| `docker image inspect <image>` | Shows full details of the image. |
| `docker image rm <image>` | Removes an image. |
| `docker image rm -f <image>` | Forcefully removes an image. |
| `docker image prune` | Removes unused (dangling) images. |

---

## 12. Save and Load Images

| Command | What it does |
|---|---|
| `docker image save <image> > myimage.tar` | Saves an image (with all layers and tags) into a tar file. |
| `docker image load < myimage.tar` | Loads an image back from the tar file. (`docker image load -i myimage.tar` also works.) |

**Save vs Export**
- `image save` works on an **image** and keeps layers, history and tags.
- `container export` works on a **container** and gives one flat filesystem without history.

---

## 13. Dockerfile and Layered Architecture

A **Dockerfile** is a text file with instructions to build an image. The file must be named `Dockerfile` (default name).

```bash
vi Dockerfile
docker image build -t myubuntu:1 .
docker container run -it myubuntu:1
```

- `-t name:tag` gives the image a name and tag.
- The `.` at the end means "use the Dockerfile in the current directory".

**Layers:** each instruction creates one layer. Docker caches layers, so when you edit the Dockerfile, change the **lower** lines rather than the upper ones. Unchanged top layers are reused, so builds are faster.

### Common Instructions

| Instruction | What it does |
|---|---|
| `FROM` | Sets the base image (e.g. `FROM ubuntu`). |
| `LABEL` | Adds metadata (author, version). |
| `RUN` | Runs a command while building the image (install packages etc.). |
| `ENV` | Sets an environment variable. |
| `WORKDIR` | Sets the working directory for the following instructions. |
| `COPY src dest` | Copies a file/folder from host into the image. |
| `ADD src dest` | Like COPY, but can also extract local tar files and download from URLs. |
| `USER` | Switches to a different user for the following instructions and for the container. |
| `CMD` | Default command when the container starts. |
| `ENTRYPOINT` | Fixed command that always runs when the container starts. |

### Adding a user in a Dockerfile

```dockerfile
ENV NAME=pk
RUN useradd -d /home/pk -g root -G sudo -m -p $(echo "PASS" | openssl passwd -1 -stdin) $NAME
USER $NAME
```

Creates a user with a home directory and password, then switches to that user.

### CMD example

```dockerfile
CMD ["python"]
```

Starting the container opens the Python shell.

### COPY vs ADD (homework)

- **COPY**: simple, only copies local files. Preferred.
- **ADD**: does more (auto-extracts local `.tar` files, can fetch URLs). Use only when you need that.

### CMD vs ENTRYPOINT (homework)

| | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command or arguments | Main program of the container |
| Override at `docker run` | Easily replaced by the command you type | Not replaced (your input becomes arguments) unless you use `--entrypoint` |
| Syntax | `CMD ["python"]` | `ENTRYPOINT ["python"]` |

Real-life use: `ENTRYPOINT ["nginx"]` with `CMD ["-g", "daemon off;"]` means nginx always runs, and the arguments can be changed. Another example is `docker container run --entrypoint htpasswd registry ...` which is used later in the registry auth section.

---

## 14. Docker Volumes

Containers lose data when deleted. Storage options to keep data:

| Type | Where data lives |
|---|---|
| **Volume** | Managed by Docker on the host (`/var/lib/docker/volumes`). Best for persistent data. |
| **Bind mount** | Any folder on the host that you choose. |
| **tmpfs** | In the host's RAM only, not on disk. Gone when the container stops. |

### Volume commands

| Command | What it does |
|---|---|
| `docker volume create <name>` | Creates a volume. |
| `docker volume ls` | Lists volumes. |
| `docker volume inspect <name>` | Shows details of a volume. |
| `docker volume rm <name>` | Deletes a volume. |
| `docker volume prune` | Deletes all unused volumes. |

### Example: MySQL with a volume

```bash
docker container run -d --name pkmysql -v abc:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=mypass mysql
```

- `-v abc:/var/lib/mysql` creates (if needed) volume `abc` and mounts it at MySQL's data folder, so the database survives container deletion.
- `-e` sets an environment variable.

Password options for the MySQL image:

```bash
# With a root password
docker container run --name db -e MYSQL_ROOT_PASSWORD=<password> -d mysql:<tag>

# Without a password
docker container run --name db -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -d mysql:<tag>
```

`<tag>` is the MySQL version.

---

## 15. Bind Mounts

Mount a host folder into a container.

```bash
# Short form
docker container run -it -v /home/root/bind:/tmp/test ubuntu bash

# Mount the current directory
docker container run -it -v $(pwd):/tmp/test ubuntu bash

# Long form
docker container run -it --mount type=bind,source=$(pwd),target=/tmp/ ubuntu bash
```

Files in the host folder appear inside the container at the target path. Changes on either side are visible on both.

---

## 16. Docker Networking

| Command | What it does |
|---|---|
| `docker network ls` | Lists networks. |
| `docker network inspect <name>` | Shows details of a network (connected containers, IP range). |
| `docker network create -d bridge test` | Creates a custom **bridge** network named `test`. |
| `docker container run -it --network test ubuntu bash` | Starts a container on that network. |
| `docker network connect <network> <container>` | Attaches a running container to a network. |
| `docker network disconnect <network> <container>` | Detaches a container from a network. |
| `docker network rm <network>` | Removes a network. |
| `docker network prune` | Removes all unused networks. |

**Bridge network:** the default network type. Containers on the same bridge can talk to each other, and are isolated from other networks. Custom bridge networks also let containers reach each other by name.

All network sub-commands: `connect`, `create`, `disconnect`, `inspect`, `ls`, `prune`, `rm`.

---

## 17. Private Docker Registry (Unsecured)

A registry is a server that stores images. You can run your own using the `registry` image.

```bash
# Start the registry
docker container run -d -p 5000:5000 --name myreg registry

# Check contents (empty at first)
curl 127.0.0.1:5000/v2/_catalog
# {"repositories":[]}

# Tag and push an image to it
docker image tag ubuntu 127.0.0.1:5000/ubuntuok
docker image push 127.0.0.1:5000/ubuntuok

# Check again
curl localhost:5000/v2/_catalog

# Pull from your local registry (works even without internet)
docker image pull 127.0.0.1:5000/ubuntuok
```

By default `pull` uses `docker.io`. Docker normally allows only HTTPS registries, with one exception: addresses in `127.0.0.0/8` (localhost).

### Using an insecure registry on another server

Create `daemon.json`:

```json
{
  "insecure-registries": ["reposerverip:5000"]
}
```

Then:

```bash
mv daemon.json /etc/docker/
service docker restart
```

---

## 18. Private Docker Registry (Secure, HTTPS)

**Step 1: Clean up the insecure setup**

```bash
cd /etc/docker/
rm -rf daemon.json
docker container ls
docker container stop <registry-container-id>
service docker restart
```

**Step 2: Create a self-signed certificate**

```bash
mkdir certs
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
```

When asked for **Common Name**, enter the server hostname: `repo.docker.local`. Press Enter for the other questions.

**Step 3: Tell Docker to trust the certificate**

```bash
cd /etc/docker
mkdir certs.d
cd certs.d
mkdir repo.docker.local:5000
cd
cp certs/domain.crt /etc/docker/certs.d/repo.docker.local\:5000/ca.crt
service docker restart
```

**Step 4: Run the secure registry**

```bash
docker container run -d -p 5000:5000 --name secure_registry \
  -v $(pwd)/certs/:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry
```

**Step 5: Push an image**

```bash
docker image ls
docker image tag mariadb repo.docker.local:5000/mariadb_local
docker image push repo.docker.local:5000/mariadb_local
```

This fails with "no such host" because the name `repo.docker.local` is not resolvable.

**Step 6: Fix the name resolution**

```bash
ifconfig                      # find the server IP
vim /etc/hosts                # add this line, then save with :wq
<ip>  repo.docker.local
```

Push again. It works now.

---

## 19. Registry with Basic Authentication

Adds a username and password on top of HTTPS.

**Step 1: Create the password file**

```bash
mkdir auth
docker container run --entrypoint htpasswd registry -bnB pawan pass123 > auth/htpasswd
cat auth/htpasswd
```

Flags: `-b` batch mode (password on command line), `-n` print to screen instead of a file, `-B` use bcrypt. Username is `pawan`, password is `pass123`. The file stores `username:encrypted-password`.

**Step 2: Run the registry with auth**

```bash
docker container run -d -p 5000:5000 --name registry_basic \
  -v "$(pwd)/auth":/auth \
  -v "$(pwd)/certs":/certs \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry
```

**Step 3: Test**

```bash
docker image push repo.docker.local:5000/mariadb_local
# Error: no basic auth credentials

docker login repo.docker.local:5000
# username: pawan
# password: pass123
# Login Succeeded

docker image push repo.docker.local:5000/mariadb_local
# Push success
```

---

## Quick Revision Cheat Sheet

```text
Run         docker container run -d -p 8080:80 --name web nginx
Shell in    docker container exec -it web /bin/bash
Logs        docker container logs web
Stop/Start  docker container stop web / start web
Clean up    docker container prune, docker image prune, docker volume prune, docker network prune
Build       docker image build -t myapp:1 .
Share       docker login -> docker image tag -> docker push
Persist     -v volume:/path  or  -v $(pwd):/path
Network     docker network create -d bridge test
```
