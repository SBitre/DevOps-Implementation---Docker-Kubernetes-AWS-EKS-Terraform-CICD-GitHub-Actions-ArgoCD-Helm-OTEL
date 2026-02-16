# Docker Fundamentals — Container & Image Lifecycle Management

## Overview
Docker containers are running instances of Docker images. Images are the blueprints, containers are the actual running (or stopped) processes. Managing both — creating, inspecting, stopping, removing — is the foundation of working with Docker.

---

## Images vs Containers

```
IMAGE (Blueprint)          CONTAINER (Running Instance)
┌──────────────┐          ┌──────────────────┐
│  hello-world │ ──run──► │  Container c5bc.. │
│  (10.1kB)    │          │  Status: Exited   │
│  Read-only   │          │  Name: angry_...  │
└──────────────┘          └──────────────────┘
     │                           │
     │ Stored locally            │ Created from image
     │ Pulled from Docker Hub    │ Has its own ID & name
     │ Can create many           │ Can be started/stopped
     │   containers from it      │ Must be removed before
     │                           │   image can be deleted
     └───────────────────────────┘
```

- **Image** = Template. Read-only. Downloaded from a registry (Docker Hub by default).
- **Container** = A running (or stopped) instance of an image. Writable layer on top of image.
- One image can spawn many containers. Deleting a container doesn't delete the image.

---

## Commands Breakdown

### 1. `docker images`
```bash
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    1b44b5a3e06a   4 weeks ago    10.1kB
```
**What it does:** Lists all Docker images stored locally on your machine.

| Column | Meaning |
|--------|---------|
| REPOSITORY | Name of the image (hello-world) |
| TAG | Version of the image (latest = most recent) |
| IMAGE ID | Unique identifier (1b44b5a3e06a) — shortened hash |
| CREATED | When the image was built by its creator |
| SIZE | How much disk space the image takes |

**Key point:** This image was pulled from Docker Hub (hub.docker.com). Docker Hub is the default public registry — like GitHub but for container images.

---

### 2. `docker ps`
```bash
$ docker ps
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```
**What it does:** Lists only **running** containers.

Output is empty because no containers are currently running. The hello-world container runs, prints a message, and exits immediately.

---

### 3. `docker ps -a`
```bash
$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS   NAMES
c5bc3cb8c8ad   hello-world   "/hello"   About a minute   Exited (0) About a minute           angry_burnell
```
**What it does:** Lists **all** containers — running AND stopped.

| Column | Meaning |
|--------|---------|
| CONTAINER ID | Unique ID of this container (c5bc3cb8c8ad) |
| IMAGE | Which image this container was created from |
| COMMAND | The command that ran inside the container ("/hello") |
| CREATED | When the container was created |
| STATUS | Current state — `Exited (0)` means it finished successfully (exit code 0 = no errors) |
| PORTS | Any port mappings (none for hello-world) |
| NAMES | Docker auto-generates a random name if you don't provide one (angry_burnell) |

**Key point:** `-a` flag means "all". Without it, you only see running containers. Stopped containers still exist and take space until you remove them.

---

### 4. `docker ps -aq`
```bash
$ docker ps -aq
c5bc3cb8c8ad
```
**What it does:** Lists **all** container IDs only.

| Flag | Meaning |
|------|---------|
| `-a` | All containers (running + stopped) |
| `-q` | Quiet mode — show only the container IDs, nothing else |

**Why this is useful:** You pipe this into other commands to perform bulk operations. Instead of manually typing each container ID, you get all of them at once.

---

### 5. `docker rm $(docker ps -aq)`
```bash
$ docker rm $(docker ps -aq)
c5bc3cb8c8ad
```
**What it does:** Removes ALL stopped containers in one command.

**How it works — step by step:**
```
docker rm $(docker ps -aq)
           └──────────────┘
           This runs FIRST
           Returns: c5bc3cb8c8ad

docker rm c5bc3cb8c8ad
└─────────────────────┘
This runs SECOND
Removes the container
```

`$(...)` is called command substitution — it runs the inner command first and passes the result to the outer command.

**Think of it like:** "Get me all container IDs, then delete all of them."

**Key point:** This only removes STOPPED containers. Running containers need to be stopped first (`docker stop`) or force removed (`docker rm -f`).

---

### 6. Verifying container removal
```bash
$ docker ps -aq
(empty — no containers left)

$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED      SIZE
hello-world   latest    1b44b5a3e06a   4 weeks ago  10.1kB
```
**Key point:** Container is gone but the IMAGE still exists. Removing a container doesn't remove its image. They are independent.

---

### 7. `docker images -q`
```bash
$ docker images -q
1b44b5a3e06a
```
**What it does:** Lists only image IDs (quiet mode), same concept as `docker ps -q` but for images.

---

### 8. `docker rmi $(docker images -q)`
```bash
$ docker rmi $(docker images -q)
```
**What it does:** Removes ALL images in one command.

| Command | What it removes |
|---------|----------------|
| `docker rm` | Removes **containers** |
| `docker rmi` | Removes **images** (the `i` stands for image) |

**How it works:**
```
docker rmi $(docker images -q)
            └────────────────┘
            Returns: 1b44b5a3e06a

docker rmi 1b44b5a3e06a
└──────────────────────┘
Deletes the hello-world image
```

**IMPORTANT:** You CANNOT delete an image if a container (even a stopped one) is using it. You must remove the container first, then the image. That's why we ran `docker rm` before `docker rmi`.

```
Wrong order (will fail):
  docker rmi hello-world  ❌  (container still exists)

Correct order:
  docker rm c5bc3cb8c8ad  ✅  (remove container first)
  docker rmi hello-world  ✅  (now image can be removed)
```

---

## The Full Lifecycle

```
1. PULL          docker pull hello-world
   (Download image from Docker Hub to local machine)
        │
        ▼
2. RUN           docker run hello-world
   (Create a container from the image and start it)
        │
        ▼
3. CONTAINER     docker ps -a  →  shows the container
   EXISTS        (running or stopped, takes up space)
        │
        ▼
4. REMOVE        docker rm <container-id>
   CONTAINER     (delete the container, image stays)
        │
        ▼
5. REMOVE        docker rmi <image-id>
   IMAGE         (delete the image from local machine)
```

---

## Quick Reference — Commands Used

| Command | Purpose | Mnemonic |
|---------|---------|----------|
| `docker images` | List all local images | "Show me all blueprints" |
| `docker ps` | List running containers | "Process status" (from Linux `ps`) |
| `docker ps -a` | List ALL containers (running + stopped) | `-a` = all |
| `docker ps -q` | List container IDs only | `-q` = quiet |
| `docker ps -aq` | List ALL container IDs only | Combine both flags |
| `docker rm <id>` | Remove a container | `rm` = remove |
| `docker rm $(docker ps -aq)` | Remove ALL stopped containers | Bulk delete trick |
| `docker rmi <id>` | Remove an image | `rmi` = remove image |
| `docker rmi $(docker images -q)` | Remove ALL images | Bulk delete trick |

---

## Common Mistakes
1. **Trying to remove an image before its containers** → Error. Always `docker rm` containers first, then `docker rmi` images.
2. **Forgetting `-a` with `docker ps`** → You think there are no containers but there are stopped ones hiding.
3. **Confusing `rm` and `rmi`** → `rm` = containers, `rmi` = images. The `i` matters.

---

## Key Takeaways
- Docker Hub is the default registry where images are pulled from
- Images are blueprints (read-only), containers are instances (writable)
- Stopped containers still exist until explicitly removed
- Always remove containers before removing their images
- Use `$(command)` substitution for bulk operations — this is a Bash trick, not Docker-specific
- The `-q` flag (quiet) is your friend for scripting and bulk operations