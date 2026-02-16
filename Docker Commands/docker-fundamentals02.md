# Docker — Pull from Hub, Run Containers, Build Images & Push to Docker Hub

## Part 1: Pull from Docker Hub and Run Containers

### Overview
Docker Hub (hub.docker.com) is the default public registry for Docker images. Anyone can pull public images and run them as containers. This section covers pulling images, running containers with different options, and managing the container lifecycle.

---

### Pulling Images

```bash
# Pull an image from Docker Hub
docker pull stacksimplify/retail-store-sample-ui:1.0.0
```

```
What happens:
Docker Hub (cloud)          Your EC2 (local)
┌─────────────────┐         ┌─────────────────┐
│ retail-store-    │ pull    │ retail-store-    │
│ sample-ui:1.0.0 │ ──────► │ sample-ui:1.0.0 │
│                  │         │ (617MB)          │
└─────────────────┘         └─────────────────┘
```

```bash
# Verify image is downloaded
docker images
# REPOSITORY                             TAG       IMAGE ID       CREATED        SIZE
# stacksimplify/retail-store-sample-ui   1.0.0     72232e8951c8   46 years ago   617MB
```

**Image naming convention:**
```
stacksimplify/retail-store-sample-ui:1.0.0
└────────────┘ └──────────────────────┘ └───┘
  username/     image name               tag
  org name                             (version)
```

- If no tag is specified, Docker defaults to `latest`
- Official images (like `nginx`, `python`) don't have a username prefix

---

### Running Containers

#### Basic Run (Foreground)
```bash
docker run stacksimplify/retail-store-sample-ui:1.0.0
```
- Runs in foreground (locks your terminal)
- Press `Ctrl+C` to stop
- Not useful for servers/apps that need to keep running

#### Detached Mode (-d)
```bash
docker run -d stacksimplify/retail-store-sample-ui:1.0.0
```
- `-d` = detached (runs in background)
- Returns the container ID
- Your terminal stays free

#### With Port Mapping (-p)
```bash
docker run -d -p 8888:8080 stacksimplify/retail-store-sample-ui:1.0.0
```

```
Port Mapping Explained:

-p 8888:8080
   │      │
   │      └── Container port (app listens on 8080 INSIDE the container)
   │
   └── Host port (you access from browser on 8888)

Browser → http://<EC2-IP>:8888 → EC2 host port 8888 → Container port 8080 → App
```

**IMPORTANT:** The container port (right side) is defined by the application. You can't change it. The host port (left side) is your choice.

```bash
# Run same app on different host ports
docker run -d -p 8888:8080 stacksimplify/retail-store-sample-ui:1.0.0
docker run -d -p 9999:8080 stacksimplify/retail-store-sample-ui:1.0.0

# Now accessible at both:
# http://<IP>:8888
# http://<IP>:9999
```

#### With Custom Name (--name)
```bash
docker run --name myapp1 -p 8888:8080 -d stacksimplify/retail-store-sample-ui:1.0.0
```
- `--name myapp1` gives the container a meaningful name instead of a random one (like "angry_burnell")
- Easier to manage: `docker stop myapp1` instead of `docker stop 64f1b2af517a`

---

### Accessing the Running App

**Requirements for accessing from browser:**
1. Container must be running (`docker ps` shows it)
2. Port mapping must be correct (`-p hostPort:containerPort`)
3. EC2 Security Group must allow inbound traffic on the host port

```
Security Group Inbound Rule:
Type: Custom TCP
Port: 8888
Source: 0.0.0.0/0 (Anywhere)
```

**URL format:** `http://<EC2-Public-IP>:<host-port>`
- Use `http` NOT `https`
- Always include the port number

---

### Container Lifecycle Commands

```bash
# List running containers
docker ps

# List ALL containers (running + stopped)
docker ps -a

# Stop a running container
docker stop myapp1

# Start a stopped container
docker start myapp1

# Restart a container
docker restart myapp1

# View container logs
docker logs myapp1

# Follow logs in real-time (like tail -f)
docker logs -f myapp1

# Execute a command inside a running container
docker exec -it myapp1 /bin/bash
# -i = interactive (keep STDIN open)
# -t = allocate a pseudo-TTY (terminal)
# This drops you INTO the container's Linux shell

# Inspect container details (JSON output)
docker inspect myapp1

# Remove a stopped container
docker rm myapp1

# Force remove a running container (stop + remove)
docker rm -f myapp1
```

---

### Container Lifecycle Diagram

```
docker run
    │
    ▼
┌─────────┐  docker stop   ┌─────────┐  docker rm   ┌─────────┐
│ RUNNING │ ──────────────► │ STOPPED │ ────────────► │ REMOVED │
│         │                 │         │               │(gone)   │
│         │ ◄────────────── │         │               │         │
└─────────┘  docker start   └─────────┘               └─────────┘
    │
    │ docker rm -f (force)
    ▼
┌─────────┐
│ REMOVED │
└─────────┘
```

---

## Part 2: Build Docker Image and Push to Docker Hub

### Overview
Instead of just pulling other people's images, you can build your own image from your application code using a Dockerfile, and push it to Docker Hub for others (or yourself) to use.

---

### The Dockerfile

A Dockerfile is a text file with instructions to build an image. Each instruction creates a layer in the image.

```dockerfile
# Example Dockerfile for a simple Nginx app
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

```
How a Dockerfile becomes a running app:

Dockerfile          Docker Image         Container
┌──────────┐  build  ┌──────────┐  run  ┌──────────┐
│ FROM      │ ──────► │          │ ────► │  Running  │
│ COPY      │         │ my-app   │       │  App      │
│ EXPOSE    │         │ :1.0.0   │       │          │
│ CMD       │         │ (150MB)  │       │          │
└──────────┘         └──────────┘       └──────────┘
 (Recipe)            (Blueprint)        (Instance)
```

---

### Building an Image

```bash
# Build an image from a Dockerfile in the current directory
docker build -t <dockerhub-username>/<image-name>:<tag> .
```

```bash
# Example
docker build -t sbitre/my-web-app:1.0.0 .
```

**Breaking down the command:**
```
docker build -t sbitre/my-web-app:1.0.0 .
              │  │       │          │     │
              │  │       │          │     └── Build context (current directory)
              │  │       │          └── Tag/version
              │  │       └── Image name
              │  └── Docker Hub username
              └── -t = tag (name:version)
```

**The `.` at the end is the build context** — it tells Docker "look in the current directory for the Dockerfile and any files it needs."

```bash
# List images to verify build
docker images
# REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
# sbitre/my-web-app     1.0.0     abc123def456   5 seconds ago    150MB
```

---

### Testing the Image Locally

```bash
# Run your newly built image
docker run --name mytest -p 8080:80 -d sbitre/my-web-app:1.0.0

# Verify it's running
docker ps

# Test it
curl http://localhost:8080
```

Always test locally before pushing to Docker Hub.

---

### Pushing to Docker Hub

```bash
# Step 1: Login to Docker Hub
docker login
# Enter your Docker Hub username and password/token

# Step 2: Push the image
docker push sbitre/my-web-app:1.0.0
```

```
What happens during push:

Your EC2 (local)             Docker Hub (cloud)
┌─────────────────┐         ┌─────────────────┐
│ sbitre/          │ push    │ sbitre/          │
│ my-web-app:1.0.0 │ ──────► │ my-web-app:1.0.0 │
│                   │         │                   │
└─────────────────┘         └─────────────────┘
                             Now anyone can:
                             docker pull sbitre/my-web-app:1.0.0
```

**Image naming matters for push:**
```
sbitre/my-web-app:1.0.0
└──────┘
Must match your Docker Hub username
Otherwise push will be rejected
```

---

### Tagging Images

```bash
# Tag an existing image with a new name/version
docker tag sbitre/my-web-app:1.0.0 sbitre/my-web-app:latest

# Now you have two tags pointing to the same image
docker images
# REPOSITORY            TAG       IMAGE ID
# sbitre/my-web-app     1.0.0     abc123def456
# sbitre/my-web-app     latest    abc123def456   ← Same IMAGE ID

# Push both tags
docker push sbitre/my-web-app:1.0.0
docker push sbitre/my-web-app:latest
```

**Why tag with both version and latest?**
- Version tag (`1.0.0`) — for production, you want to pin exact versions
- `latest` tag — for convenience, always points to the newest version

---

### The Complete Build & Push Flow

```
1. Write your application code
        │
        ▼
2. Write a Dockerfile
        │
        ▼
3. docker build -t username/app:version .
   (Creates image locally)
        │
        ▼
4. docker run -p 8080:80 -d username/app:version
   (Test locally — does it work?)
        │
        ▼
5. docker login
   (Authenticate with Docker Hub)
        │
        ▼
6. docker push username/app:version
   (Upload to Docker Hub)
        │
        ▼
7. Anyone anywhere can now:
   docker pull username/app:version
   docker run username/app:version
```

---

## Quick Reference — All Commands

### Image Commands
| Command | Purpose |
|---------|---------|
| `docker pull <image>` | Download image from registry |
| `docker images` | List all local images |
| `docker images -q` | List only image IDs |
| `docker build -t <name>:<tag> .` | Build image from Dockerfile |
| `docker tag <source> <target>` | Create a new tag for an image |
| `docker push <image>` | Push image to registry |
| `docker rmi <image>` | Remove an image |
| `docker rmi $(docker images -q)` | Remove ALL images |

### Container Commands
| Command | Purpose |
|---------|---------|
| `docker run -d -p H:C --name N <image>` | Run container (detached, port mapped, named) |
| `docker ps` | List running containers |
| `docker ps -a` | List ALL containers |
| `docker ps -aq` | List all container IDs |
| `docker stop <name/id>` | Stop a container |
| `docker start <name/id>` | Start a stopped container |
| `docker restart <name/id>` | Restart a container |
| `docker logs <name/id>` | View container logs |
| `docker logs -f <name/id>` | Follow logs in real-time |
| `docker exec -it <name/id> /bin/bash` | Get shell inside container |
| `docker inspect <name/id>` | Detailed container info (JSON) |
| `docker rm <name/id>` | Remove stopped container |
| `docker rm -f <name/id>` | Force remove running container |
| `docker rm $(docker ps -aq)` | Remove ALL containers |

### Authentication
| Command | Purpose |
|---------|---------|
| `docker login` | Login to Docker Hub |
| `docker logout` | Logout from Docker Hub |

---

## Key Takeaways

1. **Docker Hub is like GitHub for containers** — public registry where anyone can push/pull images
2. **Port mapping (-p) is essential** — without it, your app runs but nobody can reach it
3. **Host port (left) is your choice, container port (right) is fixed** by the application
4. **Security groups are the firewall** — even correct port mapping won't work without allowing the port in AWS
5. **Always use `http` not `https`** when accessing apps on custom ports (unless SSL is configured)
6. **Name your containers (--name)** — makes management much easier than random names
7. **Image naming convention matters** — `username/image:tag` must match your Docker Hub username for push to work
8. **Test locally before pushing** — always `docker run` and verify before `docker push`
9. **Tag with version AND latest** — versions for production safety, latest for convenience
10. **The Dockerfile is the recipe** — it defines what goes into your image, making builds reproducible