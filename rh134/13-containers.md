# Chapter 13. Run Containers

Goal: get, run and manage simple lightweight services as containers on a single RHEL server.

Sections:

1. Introduction to Containers
2. Creating Containers with Podman
3. Container Image Registries
4. Managing the Container Lifecycle

## 1. Introduction to Containers

- A container = an isolated process with its own libraries and dependencies. Small, starts and stops fast.
- Image layers are immutable. The engine adds one writable layer on top. Containers are ephemeral: the writable layer is removed with the container.
- Linux kernel features: namespaces (isolation), cgroups (CPU and memory limits), SELinux and seccomp (security).
- OCI (Open Container Initiative) sets the standards: `image-spec` (images), `runtime-spec` (instances).
- Image = class, container instance = object. One image can create many instances on many hosts.

|                | VM                          | Container                 |
| -------------- | --------------------------- | ------------------------- |
| Managed by     | hypervisor (KVM, VMware...) | container engine (Podman) |
| Virtualization | whole environment           | only relevant parts       |
| Size           | gigabytes                   | megabytes                 |
| Portability    | usually same hypervisor     | any OCI-compliant engine  |

- Use a VM when you need dedicated hardware, a non-Linux OS or a different kernel.
- At scale: Kubernetes, Red Hat OpenShift Container Platform (RHOCP, manages containers and VMs).
- Multi-container apps: several containers (for example DB + web API) or replicas for HA. Tools: Podman Pods, compose-spec.

### Dev use

```
same image on laptop, CI and production   # no "works on my machine": libraries are inside the image
```

- Main benefit for developers: the same Python or Node version everywhere. Not from the course.

## 2. Creating Containers with Podman

- Podman manages OCI containers and images locally. It is daemonless (no always-running daemon, no single point of failure). Interfaces: CLI, REST API, Podman Desktop.

```bash
podman pull registry.redhat.io/rhel9/rhel-guest-image:9.4   # NAME:VERSION
podman images                                               # local images
podman run registry.redhat.io/rhel9/rhel-guest-image:9.4 echo 'Red Hat'
podman ps                                                   # running containers
podman ps --all                                             # also stopped ones
```

- `podman run` pulls the image automatically if it is missing.
- The container stops when its command ends, but it is not removed.

| Option          | Meaning                            |
| --------------- | ---------------------------------- |
| `--rm`          | remove the container when it exits |
| `--name NAME`   | set a name (otherwise random)      |
| `-p 8080:8080`  | map port `host:container`          |
| `-d`            | run in the background              |
| `-e NAME=value` | pass an environment variable       |

```bash
podman run -d --name web -p 8080:8080 registry.access.redhat.com/ubi9/httpd-24:latest   # open localhost:8080
podman run -e NAME='Red Hat' registry.redhat.io/rhel9/rhel-guest-image:9.4 printenv NAME
```

- Container ID: short (12 characters) or long (64, from `--format=json`), or the name.
- Environment variables = safe way to pass environment-specific settings (for example the database host).
- Podman Desktop: GUI for local containers (also supports Docker). It complements the CLI. Not shipped with RHEL.

### Dev use

```bash
podman run -d --name db -p 5432:5432 -e POSTGRES_PASSWORD=dev docker.io/library/postgres:16   # local database for development
```

- Quick throwaway database for a project. Not from the course.

## 3. Container Image Registries

- A registry stores images to share them (Quay.io, Red Hat Registry, Docker Hub, Amazon ECR).
- Red Hat Registry: `registry.access.redhat.com` (no authentication), `registry.redhat.io` (authentication). Search: Red Hat Ecosystem Catalog (`catalog.redhat.com`).
- UBI (Universal Base Image): free OCI image based on RHEL, no subscription needed. Full support only on Red Hat platforms. Examples: `ubi9/python-312`, `ubi9/nodejs-18`.
- Quay.io: for your own images (public are free, private are paid).
- Image name: `registry.access.redhat.com/ubi9/nodejs-18:latest` = registry / organization / repository : tag.
- Short names are searched in `/etc/containers/registries.conf`. Always use the fully qualified name (a short name can pull an unsupported or malicious image).

```
unqualified-search-registries = ['registry.redhat.io', 'docker.io']

[[registry]]
location="docker.io"
blocked=true
```

Skopeo: inspect and copy images without local storage. Format `transport:image` (`docker://`, `dir:`, `oci:`).

```bash
skopeo inspect docker://registry.access.redhat.com/ubi9/nodejs-18
skopeo copy docker://registry.access.redhat.com/ubi9/nodejs-18 docker://quay.io/myuser/nodejs-18
```

Authentication:

```bash
podman login registry.redhat.io     # password is not shown while typing
```

- Credentials are stored in `${XDG_RUNTIME_DIR}/containers/auth.json` as base64 (not encrypted). Skopeo uses the same file.

### Dev use

```bash
podman pull registry.access.redhat.com/ubi9/nodejs-18:latest   # fully qualified name, no login needed
```

- Use fully qualified names in Dockerfiles, Containerfiles and CI. Not from the course.

## 4. Managing the Container Lifecycle

| Command                                            | Use                                           |
| -------------------------------------------------- | --------------------------------------------- |
| `podman ps` / `podman ps -a`                       | running / all containers                      |
| `podman inspect NAME`                              | full info (JSON)                              |
| `podman inspect --format='{{.State.Status}}' NAME` | one field (Go template)                       |
| `podman container logs NAME`                       | container logs                                |
| `podman stop NAME`                                 | SIGTERM, then SIGKILL after 10 s (`--time=N`) |
| `podman kill NAME`                                 | SIGKILL at once                               |
| `podman pause` / `unpause`                         | suspend (SIGSTOP) / resume                    |
| `podman restart NAME`                              | restart (or start a stopped container)        |
| `podman rm NAME`                                   | remove a stopped container                    |
| `podman rm --force` / `--all`                      | remove a running one / all stopped            |

- Stopped is not removed: it stays as `Exited` (see `podman ps -a`). Commands accept the ID or the name.

Persistent storage (container data is lost on removal):

```bash
mkdir /home/user/db_data
podman unshare chown 27:27 /home/user/db_data      # owner = mysql (UID 27) inside the container
podman run -d --name db01 -e MYSQL_USER=student -e MYSQL_PASSWORD=student \
-e MYSQL_DATABASE=dev_data -e MYSQL_ROOT_PASSWORD=redhat \
-v /home/user/db_data:/var/lib/mysql:Z \
registry.lab.example.com/rhel8/mariadb-105
```

- Rootless containers use a user namespace: container root (0) = your host user, container UID 27 = host UID 100026. Check: `podman unshare cat /proc/self/uid_map`.
- The directory needs the SELinux type `container_file_t`. The `:Z` suffix in `-v` sets it. Without it: `Permission denied` (see `podman container logs`).

Start a container on boot (systemd, rootless):

```bash
podman generate systemd --name web --files        # creates container-web.service
# move it to ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now container-web.service
loginctl enable-linger                            # run from OS boot, not only while logged in
```

### Dev use

```bash
podman run -d --name db -v ./pgdata:/var/lib/postgresql/data:Z -e POSTGRES_PASSWORD=dev docker.io/library/postgres:16
```

- Local database that keeps its data after the container is removed. Not from the course.
