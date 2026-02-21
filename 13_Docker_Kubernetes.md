# 13 — Docker & Kubernetes (Interview Ready)

> **Level:** 3-4 Years Experience | **Priority:** 🟡 Important  
> Covers: Docker Basics, Dockerfile, Compose, Kubernetes Basics, Pods, Deployments, Services

---

## 📌 Table of Contents

1. [Docker Fundamentals](#1-docker-fundamentals)
2. [Dockerfile](#2-dockerfile)
3. [Docker Commands](#3-docker-commands)
4. [Docker Compose](#4-docker-compose)
5. [Kubernetes Basics](#5-kubernetes-basics)
6. [K8s Core Objects](#6-k8s-core-objects)
7. [Interview Questions & Answers](#7-interview-questions--answers)

---

## 1. Docker Fundamentals

### What is Docker?
- **Containerization** platform — packages app + dependencies into a container
- **Containers** = lightweight, portable, isolated environments
- **Not a VM** — shares host OS kernel (no guest OS overhead)

### Docker vs VM
| Feature | Docker Container | Virtual Machine |
|---------|-----------------|-----------------|
| OS | Shares host kernel | Full guest OS |
| Size | MBs | GBs |
| Startup | Seconds | Minutes |
| Isolation | Process-level | Hardware-level |
| Performance | Near-native | Overhead |

### Docker Architecture
```
Docker Client (CLI)
    ↓ (commands)
Docker Daemon (dockerd)
    ↓ (manages)
Container ←── Image ←── Dockerfile
    ↓
Docker Registry (Docker Hub, ECR, GCR)
```

---

## 2. Dockerfile

### Spring Boot Application Dockerfile

```dockerfile
# Multi-stage build (production-ready)
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline          # cache dependencies
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Run (smaller image)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

# Non-root user for security
RUN addgroup -S spring && adduser -S spring -G spring
USER spring

EXPOSE 8080
HEALTHCHECK CMD curl -f http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Key Instructions
| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files from host |
| `RUN` | Execute command during build |
| `EXPOSE` | Document port (informational) |
| `ENV` | Set environment variable |
| `ENTRYPOINT` | Container startup command (not overridable easily) |
| `CMD` | Default args for ENTRYPOINT (overridable) |
| `VOLUME` | Create mount point |
| `ARG` | Build-time variable |

### Best Practices
```
✅ Use multi-stage builds (smaller images)
✅ Use specific tags, not :latest
✅ Run as non-root user
✅ Use .dockerignore to exclude unnecessary files
✅ Order Dockerfile layers — less changing layers first
✅ Combine RUN commands to reduce layers
❌ Don't store secrets in Dockerfile/image
```

---

## 3. Docker Commands

```bash
# Build & Run
docker build -t myapp:1.0 .                    # build image
docker run -d -p 8080:8080 --name myapp myapp:1.0  # run container
docker run -it ubuntu bash                      # interactive mode

# Container management
docker ps                                       # running containers
docker ps -a                                    # all containers
docker stop myapp                               # stop container
docker start myapp                              # start stopped container
docker rm myapp                                 # remove container
docker logs myapp                               # view logs
docker logs -f myapp                            # follow logs
docker exec -it myapp bash                      # enter running container

# Image management
docker images                                   # list images
docker rmi myapp:1.0                           # remove image
docker pull nginx:alpine                        # pull from registry
docker push myrepo/myapp:1.0                   # push to registry
docker tag myapp:1.0 myrepo/myapp:latest       # tag image

# Networking
docker network create mynetwork
docker run --network mynetwork --name db postgres

# Volumes (persistent data)
docker volume create mydata
docker run -v mydata:/var/lib/postgresql/data postgres

# Cleanup
docker system prune                             # remove unused containers, images, networks
```

---

## 4. Docker Compose

### Full Stack Application (Spring Boot + PostgreSQL + Redis)

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/mydb
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=secret
      - SPRING_REDIS_HOST=cache
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    networks:
      - backend

  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=mydb
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - backend

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - backend

volumes:
  postgres_data:

networks:
  backend:
    driver: bridge
```

```bash
docker-compose up -d              # start all services
docker-compose down               # stop and remove
docker-compose logs -f app        # follow app logs
docker-compose ps                 # list services
docker-compose build              # rebuild images
```

---

## 5. Kubernetes Basics

### What is Kubernetes?
- **Container orchestration** platform
- Manages deployment, scaling, networking of containers
- Self-healing (restarts failed containers)
- Rolling updates with zero downtime

### Architecture
```
┌─ Kubernetes Cluster ────────────────────────┐
│                                             │
│  Control Plane (Master)                     │
│  ├── API Server (kubectl talks to this)     │
│  ├── Scheduler (assigns pods to nodes)      │
│  ├── Controller Manager (desired state)     │
│  └── etcd (cluster state store)             │
│                                             │
│  Worker Nodes                               │
│  ├── Node 1                                 │
│  │   ├── kubelet (node agent)               │
│  │   ├── kube-proxy (networking)            │
│  │   ├── Pod [Container1, Container2]       │
│  │   └── Pod [Container3]                   │
│  ├── Node 2                                 │
│  │   ├── Pod [Container4]                   │
│  │   └── Pod [Container5]                   │
│  └── Node 3                                 │
└─────────────────────────────────────────────┘
```

---

## 6. K8s Core Objects

### Pod — smallest deployable unit
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp
      image: myapp:1.0
      ports:
        - containerPort: 8080
```

### Deployment — manages ReplicaSets & rolling updates
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:1.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 15
          livenessProbe:
            httpGet:
              path: /actuator/health
              port: 8080
            initialDelaySeconds: 30
```

### Service — expose pods within/outside cluster
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP      # ClusterIP | NodePort | LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

### Essential kubectl Commands
```bash
kubectl get pods                              # list pods
kubectl get deployments                       # list deployments
kubectl get services                          # list services
kubectl describe pod myapp-pod                # detailed info
kubectl logs myapp-pod                        # view logs
kubectl exec -it myapp-pod -- bash            # enter pod
kubectl apply -f deployment.yaml              # apply config
kubectl scale deployment myapp --replicas=5   # scale
kubectl rollout status deployment/myapp       # check rollout
kubectl rollout undo deployment/myapp         # rollback
```

---

## 7. Interview Questions & Answers

### Q1: Docker image vs container?
**A:** Image is a read-only template (blueprint). Container is a running instance of an image. You can create multiple containers from one image. Image = class, Container = object.

### Q2: What is multi-stage build?
**A:** Dockerfile with multiple `FROM` stages. First stage builds the app (includes build tools), second stage copies only the built artifact into a minimal runtime image. Result: much smaller production images.

### Q3: ENTRYPOINT vs CMD?
**A:** `ENTRYPOINT` defines the main command (not easily overridden). `CMD` provides default arguments to ENTRYPOINT (overridable at runtime). Use ENTRYPOINT for the executable, CMD for default args.

### Q4: What is a Kubernetes Pod?
**A:** Smallest deployable unit. Contains one or more tightly coupled containers sharing network (localhost) and storage. Usually one app container per pod. Pods are ephemeral — managed by Deployments.

### Q5: How does Kubernetes achieve zero-downtime deployment?
**A:** Rolling update strategy: gradually replaces old pods with new ones. Readiness probes ensure new pods are ready before receiving traffic. Old pods are terminated only after new ones are healthy.

### Q6: ClusterIP vs NodePort vs LoadBalancer?
**A:** **ClusterIP** — internal only (default). **NodePort** — exposes on each node's IP at a static port. **LoadBalancer** — provisions external cloud LB (AWS ALB/NLB). Use LoadBalancer for production, ClusterIP for internal services.

---

> **Next:** [14_Unix_Linux_Commands.md](14_Unix_Linux_Commands.md) →
