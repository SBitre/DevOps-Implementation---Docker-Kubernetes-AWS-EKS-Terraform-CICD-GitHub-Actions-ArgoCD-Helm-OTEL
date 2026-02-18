# Dockerfile — Multi-Stage Builds for Production

## The One Thing You Need to Understand

A Dockerfile has TWO stages because building an app and running an app need different tools.

```
Think of it like cooking:

KITCHEN (Build Stage)              DINING TABLE (Package Stage)
┌────────────────────┐             ┌────────────────────┐
│ Knives             │             │ Plate              │
│ Cutting board      │             │ Fork               │
│ Stove              │             │ The finished food  │
│ Pots and pans      │             │                    │
│ Raw ingredients    │             │ Nothing else.      │
│ Spices             │             │ Clean and minimal. │
│ THE FINISHED FOOD  │             │                    │
└────────────────────┘             └────────────────────┘

You don't bring the entire kitchen to the dining table.
You only bring the finished food.

Same with Docker:
You don't ship Maven, source code, and build tools to production.
You only ship the compiled app (JAR file).
```

---

## Build Stage vs Package Stage

### Build Stage — The Kitchen

```dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env
```

**Purpose:** Compile your source code into a runnable application.

**What gets installed:**
- Maven (build tool — like a compiler for Java)
- Java JDK (to compile Java code)
- Source code (your actual application code)

**What it produces:**
- `app.jar` — the compiled application (one single file)

**What happens to this stage after build?**
- It gets THROWN AWAY. None of this exists in the final image.

```
Build Stage installs:        Final image contains:
├── Maven (200MB)            NONE OF THIS
├── Java JDK (400MB)         NONE OF THIS  
├── Source code (5MB)         NONE OF THIS
├── Dependencies (100MB)      NONE OF THIS
└── app.jar (30MB)           ✅ ONLY THIS GOES TO FINAL IMAGE
```

### Package Stage — The Dining Table

```dockerfile
FROM public.ecr.aws/amazonlinux/amazonlinux:2023
```

**Purpose:** Run the compiled application with minimum footprint.

**What gets installed:**
- Java Runtime only (NOT the full JDK — just enough to run, not compile)
- A non-root user (security)
- curl (for health checks)

**What gets copied from build stage:**
```dockerfile
COPY --from=build-env /app.jar .
```
Just the JAR file. Nothing else. That `--from=build-env` is the magic — it reaches back into the build stage and grabs only what you need.

---

## Why Multi-Stage? The Size Difference

```
Single Stage (everything in one):
┌─────────────────────────────┐
│ Maven          200MB        │
│ Java JDK       400MB        │
│ Source code       5MB        │
│ Dependencies   100MB        │
│ app.jar         30MB        │
│                              │
│ TOTAL: ~735MB               │  ← Shipped to production
└─────────────────────────────┘

Multi-Stage (build + package):
┌─────────────────────────────┐
│ Java Runtime   200MB        │
│ curl            10MB        │
│ app.jar         30MB        │
│                              │
│ TOTAL: ~240MB               │  ← Shipped to production
└─────────────────────────────┘

3x smaller. Faster to pull. Faster to deploy. Less attack surface.
```

---

## The Dockerfile Line by Line

### Build Stage

```dockerfile
# Start with Amazon Linux as base, name this stage "build-env"
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env
```
`AS build-env` gives this stage a name so we can reference it later.

```dockerfile
# Install Maven (build tool) and Java (compiler)
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which tar gzip \
    && dnf clean all
```
`install_weak_deps=False` = don't install unnecessary extras (keeps image smaller)
`dnf clean all` = delete package manager cache (saves space)

```dockerfile
# Copy dependency files FIRST (for caching)
COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

# Download all dependencies
RUN ./mvnw dependency:go-offline -B -q
```
**Why copy pom.xml before source code?** Docker layer caching. Dependencies rarely change but source code changes often. By copying dependencies first, Docker caches that layer and doesn't re-download them every time you change code.

```dockerfile
# NOW copy source code
COPY ./src ./src

# Build the application
RUN ./mvnw -DskipTests package -q && \
    mv /target/ui-0.0.1-SNAPSHOT.jar /app.jar
```
Compiles everything into one file: `app.jar`

### Package Stage

```dockerfile
# Fresh start — clean base image (NOT build-env)
FROM public.ecr.aws/amazonlinux/amazonlinux:2023
```
This is a BRAND NEW image. Nothing from build stage exists here.

```dockerfile
# Install ONLY Java runtime (not Maven, not JDK)
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    && dnf clean all
```

```dockerfile
# Create a non-root user (SECURITY)
RUN useradd --home "/app" --create-home \
    --user-group --uid "1000" "appuser"
```
**Why?** If someone hacks into your container, they're `appuser` not `root`. They can't install software, modify system files, or escape the container easily.

```dockerfile
# Switch to non-root user
USER appuser

# Copy ONLY the JAR from build stage
COPY --chown=appuser:appuser --from=build-env /app.jar .
```
`--from=build-env` = grab this file from the build stage
`--chown=appuser:appuser` = make the non-root user the owner

```dockerfile
# Tell Docker this app listens on port 8080
EXPOSE 8080

# Run the application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
```

---

## .dockerignore — Keep Builds Clean

```
Dockerfile
docker-compose.yml
target/
.idea/
scripts/
chart/
```

Same concept as `.gitignore`. Tells Docker "don't include these files when building." Without this, Docker copies EVERYTHING in the directory into the build context — including stuff you don't need.

---

## Docker Layer Caching — Why Builds Get Faster

```
First build:  110 seconds (downloads everything from scratch)
Second build:   0.3 seconds (everything is cached, nothing changed)
```

**How it works:**

Docker builds images in layers. Each instruction = one layer.

```
Layer 1: FROM amazonlinux        ← cached after first build
Layer 2: RUN dnf install maven   ← cached (didn't change)
Layer 3: COPY pom.xml            ← cached (file didn't change)
Layer 4: RUN mvnw dependency     ← cached (pom.xml didn't change)
Layer 5: COPY ./src              ← REBUILDS (you changed source code)
Layer 6: RUN mvnw package        ← REBUILDS (source changed)
```

**Key rule:** The moment one layer changes, ALL layers after it rebuild. That's why we copy `pom.xml` before `src/` — dependencies are cached, only code changes trigger a rebuild.

```
Smart order:                    Dumb order:
COPY pom.xml          ✅       COPY . .              ❌
RUN install deps               RUN install deps
COPY src/                      (Any file change = rebuild 
RUN build                       EVERYTHING including deps)
```

---

## Cleanup Commands

| Command | What it does | When to use |
|---------|-------------|-------------|
| `docker stop <name>` | Stop a running container | Done testing |
| `docker rm <name>` | Remove a stopped container | After stopping |
| `docker rmi <image>` | Remove an image | Don't need it anymore |
| `docker builder prune` | Remove build cache | Disk space getting full |
| `docker builder prune --all` | Remove ALL build cache | Full cleanup |
| `docker system prune` | Remove unused containers, networks, images | Weekly cleanup |
| `docker system prune -a --volumes -f` | NUCLEAR OPTION — removes everything | Only when you want a fresh start |
| `docker build --no-cache -t name .` | Rebuild ignoring all cache | When cache is causing issues |

**Be careful with `docker system prune -a`** — it deletes ALL unused images. If you have images you still need but aren't running, they'll be gone.

---

## Key Takeaways

1. **Multi-stage builds = smaller, safer images.** Build tools don't ship to production. Only the compiled app does.

2. **`--from=build-env`** is the bridge between stages. It copies specific files from the build stage to the package stage.

3. **Non-root user is a security requirement.** Never run production containers as root. Always create an `appuser`.

4. **Layer caching makes builds fast.** Put things that change rarely (dependencies) before things that change often (source code).

5. **`.dockerignore` keeps builds clean.** Same idea as `.gitignore` — exclude unnecessary files from the build.

6. **First build is slow, subsequent builds are fast.** Docker caches unchanged layers. This is why the second build took 0.3 seconds vs 110 seconds.

7. **You don't need to understand Java to Dockerize a Java app.** You need to understand the Dockerfile instructions and the build → package → run flow. That's DevOps.