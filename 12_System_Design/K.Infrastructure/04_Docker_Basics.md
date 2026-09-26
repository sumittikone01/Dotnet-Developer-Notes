# 🐳 Docker Basics

## 📌 What is it?

**Docker** is a platform for packaging an application and **everything it needs to run** (code, runtime, libraries, system tools, config) into a single portable unit called a **container**.

## 🤔 Why do we need it?

> "It works on my machine" 🙃 — Docker exists to kill this problem forever.

- **Consistency**: Same environment runs identically on your laptop, a teammate's laptop, CI pipeline, and production.
- **Isolation**: Each container runs independently — no dependency conflicts between apps sharing a host.
- **Portability**: Runs anywhere Docker is installed — any cloud, any OS.
- **Fast startup**: Containers start in seconds (vs minutes for a full VM).
- **Efficient resource use**: Multiple containers share the host OS kernel instead of each needing a full guest OS.

## 🌍 Real-world analogy

A **shipping container** 📦 (literally where Docker gets its name/logo from). Before standardized shipping containers, cargo was loaded by hand in all shapes and sizes — slow, inconsistent, and error-prone. Standardized containers meant *any* ship, train, or truck could carry *any* cargo the same way, everywhere in the world. Docker does this for software.

## 📊 Container vs Virtual Machine

| Aspect                      | Container             | Virtual Machine                |
| --------------------------- | --------------------- | ------------------------------ |
| **OS**                | Shares host OS kernel | Full separate guest OS per VM  |
| **Size**              | MBs                   | GBs                            |
| **Startup time**      | Seconds               | Minutes                        |
| **Isolation level**   | Process-level         | Full hardware-level (stronger) |
| **Resource overhead** | Low                   | High                           |

### 🖼 Diagram

```
VIRTUAL MACHINES:                    CONTAINERS:
┌───────┬───────┬───────┐            ┌───────┬───────┬───────┐
│ App A │ App B │ App C │            │ App A │ App B │ App C │
├───────┼───────┼───────┤            ├───────┴───────┴───────┤
│Guest  │Guest  │Guest  │            │     Docker Engine      │
│OS     │OS     │OS     │            ├────────────────────────┤
├───────┴───────┴───────┤            │      Host OS Kernel    │
│      Hypervisor         │           └────────────────────────┘
├────────────────────────┤
│      Host OS Kernel     │
└────────────────────────┘
```

## ⚙️ Core Concepts

| Term                     | Meaning                                                                    |
| ------------------------ | -------------------------------------------------------------------------- |
| **Image**          | A read-only blueprint/template for a container (like a class)              |
| **Container**      | A running instance of an image (like an object)                            |
| **Dockerfile**     | Script defining how to build an image, step by step                        |
| **Registry**       | Storage for images (e.g. Docker Hub, GitHub Container Registry)            |
| **Volume**         | Persistent storage that survives container restarts/removal                |
| **Docker Compose** | Tool to define & run multi-container apps together (e.g. app + DB + cache) |

## 💻 Example Dockerfile (ASP.NET Core app)

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

### Common Commands

```bash
docker build -t myapp:1.0 .        # build image from Dockerfile
docker run -p 8080:80 myapp:1.0    # run container, map host:container ports
docker ps                          # list running containers
docker logs <container_id>         # view container logs
docker exec -it <container_id> sh  # open a shell inside running container
```

### Example docker-compose.yml (App + DB together)

```yaml
version: '3.8'
services:
  webapp:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```

## 🚨 Common Mistakes

- ❌ Storing data inside the container without a volume — data is **lost** when the container is removed.
- ❌ Running containers as `root` — security risk.
- ❌ Building huge images by not using multi-stage builds (shipping SDK + build tools into production image unnecessarily).
- ❌ Hardcoding config/secrets into the image instead of using environment variables.

## 💡 Best Practices

- Use **multi-stage builds** to keep production images small (build tools stay in the build stage only, as shown above).
- Use `.dockerignore` to avoid copying `node_modules`, `.git`, etc. into the image.
- Store persistent data in **volumes**, never inside the container filesystem.
- Pass configuration via **environment variables**, not hardcoded values.
- Keep one process per container (single responsibility, mirrors Microservices philosophy — see `02_Microservices.md`).

## 🎤 Interview Questions

1. What's the difference between a Docker image and a Docker container?
2. Why are containers more lightweight than virtual machines?
3. What is a multi-stage Dockerfile and why is it useful?
4. How does Docker handle persistent data, given containers are ephemeral by design?

## 📝 30-second Revision Cheat Sheet

- Docker = package app + dependencies into a portable container.
- Image = blueprint, Container = running instance.
- Much lighter than VMs — shares host OS kernel.
- Use volumes for persistent data, multi-stage builds for small images.
- Docker Compose runs multi-container setups (app + DB + cache) together.
