# Multi-Stage Build

Now we will decrease the image size.

- Multi-stage builds introduce **multiple stages** in a single Dockerfile. Each stage has a specific purpose.
- We use **one Dockerfile**, but it contains multiple stages.
- One stage is used as the **builder** to build the application and install the required dependencies.
- Another stage is used as the **final image** to run the application.
- We can copy only the required files from the builder stage to the final stage.
- Build tools and unnecessary files remain in the builder stage and are not copied to the final image.
- This reduces the image size by removing unnecessary installations and build files.

**Flow:**

```text
Builder Stage
    ↓
Build application + Install dependencies
    ↓
Copy only required files
    ↓
Final Image
    ↓
Small, clean and optimized image
```

# Build and Deploy the Catalogue Service

Go to the Catalogue project directory:

```bash
cd catalogue
```

Build the Docker image:

```bash
docker build -t lakshmi1092/catalogue:v1 .
```

Verify the image:

```bash
docker images
```

Go back to the project root directory:

```bash
cd ..
```

Start (or recreate) the services using Docker Compose:

```bash
docker compose up -d
```

> **Note:** After rebuilding the image, run `docker compose up -d` so Docker Compose recreates the container with the latest image.


# Optimized Docker Layers

Go to the User project:

```bash
cd user
```

Build the image without using the cache:

```bash
DOCKER_BUILDKIT=0 docker build --no-cache -t lakshmi1092/user:v1 .
```

> `DOCKER_BUILDKIT=0` shows the traditional Docker build output with intermediate containers.

---

## Output Flow

```text
FROM node:20          (1st instruction)
        ↓
Pull the base image
        ↓
Create an intermediate container
        ↓
Execute the instruction
        ↓
Create Image Layer 1

WORKDIR /opt/server   (2nd instruction)
        ↓
Create an intermediate container from Image Layer 1
        ↓
Execute the instruction
        ↓
Create Image Layer 2

COPY package.json .   (3rd instruction)
        ↓
Create an intermediate container from Image Layer 2
        ↓
Execute the instruction
        ↓
Create Image Layer 3

RUN npm install       (4th instruction)
        ↓
Create an intermediate container from Image Layer 3
        ↓
Execute the instruction
        ↓
Create Image Layer 4

...same process continues for the remaining instructions...

        ↓
Final Docker Image
```

### Note

- Every Dockerfile instruction creates a **new image layer**.
- Docker creates a temporary (**intermediate**) container to execute each instruction.
- After executing the instruction, Docker saves the changes as a new image layer.
- This process continues until the final image is created.
- After the build is completed, all intermediate containers are automatically deleted.

---

## Push the Image

```bash
docker push lakshmi1092/user:v1
```

If you push the image again:

- Docker does **not** push the entire image.
- It checks all the image layers.
- Only the **new or modified layers** are pushed to Docker Hub.
- Existing layers are skipped, making the push faster.
  

OUTPUT:
---------
FROM node:20(1st intrscution) ----> pull ----> from this it will create a container(intermediate contnaer) ---> on top of this it will run second command (WORKDIR /opt/server/) -----> from this again it creates the image (c10b18d0862b) ----> from this image again it run the container(63f71edeecde) ------> same process for remaing 
# Docker Best Practices

- Use **minimal and official images**.
- Use **multi-stage builds** to reduce image size.
- Optimize Docker layers by combining `RUN` commands wherever possible.
- Run containers as a **non-root user**.
- Use **custom bridge networks** for container communication.
- Use **Docker volumes** for stateful applications.
- Prefer `COPY` over `ADD` unless additional `ADD` features are required.
- Use a `.dockerignore` file to exclude unnecessary files from the build context.
- Implement **health checks** to monitor container health.
- Limit CPU and memory resources for containers.
- Store sensitive information (passwords, API keys, etc.) in a **Secrets Manager** instead of hardcoding them.
- Use persistent volumes for databases and other stateful services.

---

## Build and Push Catalogue Image

```bash
cd catalogue
```

```bash
docker build -t lakshmi1092/catalogue:v1 .
```

```bash
docker login -u lakshmi1092
```

```bash
docker push lakshmi1092/catalogue:v1
```

```bash
docker compose up -d
```

Verify:

```bash
docker ps
```

```bash
docker images
```

---

## Build and Push Multiple Images

```bash
for i in cart catalogue user
do
    cd "$i"
    docker build -t lakshmi1092/$i:v1 .
    docker push lakshmi1092/$i:v1
    cd ..
done
```

Verify:

```bash
docker images
```

Recreate the containers:

```bash
docker compose up -d
```

Restart the frontend:

```bash
docker restart frontend
```

---

# Java Build Process

```text
Source Code
      ↓
Compile
      ↓
Bytecode (.class files)
      ↓
Run the Application
```

### JDK (Java Development Kit)

- Used to develop Java applications.
- Contains the Java compiler (`javac`) and other development tools.
- Required to compile **source code** into **bytecode**.

### JRE (Java Runtime Environment)

- Used to run Java applications.
- Executes the compiled bytecode.
- Does not contain development tools like the Java compiler.

### Maven

- Maven is the build tool used in Java projects.
- It manages dependencies.
- Compiles the source code.
- Runs tests.
- Packages the application into a **JAR** or **WAR** file.
  

# Docker Architecture

Docker architecture consists of three main components:

- **Client**
- **Docker Host**
- **Registry (Repository)**

---

## Client

- The client is the **Docker CLI (Command Line Interface)**.
- We use the Docker CLI to execute Docker commands such as `docker build`, `docker run`, `docker ps`, etc.

---

## Docker Host

- The Docker Host is the machine where Docker is installed.
- It runs the **Docker Daemon (`dockerd`)**, which continuously listens for Docker commands.
- The Docker Daemon is responsible for:
  - Building images.
  - Creating containers.
  - Managing Docker networks.
  - Managing Docker volumes.

---

## Registry (Repository)

- Stores Docker images.
- Images can be stored in:
  - **Local Repository** (available on the Docker Host).
  - **Central Repository** (Docker Hub or a private registry).

---

# What Happens When We Run?

```bash
docker run nginx
```

1. Docker checks whether the **nginx image** is available locally.
2. If the image exists, Docker creates and starts the container.
3. If the image does not exist, Docker pulls it from the Docker registry (Docker Hub by default), creates the container, and displays the output on the client.
4. During container creation, Docker also configures networking and volumes (if specified).

---

# Disadvantages of Docker

## Auto Scaling

- Docker does not provide automatic auto-scaling by default.
- Containers are not automatically increased or decreased based on traffic.

---

## Load Balancing

- Docker does not include a built-in load balancer.
- Traffic is not automatically distributed across multiple containers.

---

## Reliability (Self-Healing)

- If a container crashes, Docker does not automatically recreate or restart it (unless a restart policy is configured).
- Docker does not provide self-healing by default.

---

## Docker Host Failure

- If the Docker Host crashes, all the containers running on that host become unavailable.

---

## Storage

- By default, Docker-managed volumes are stored on the same Docker Host.
- If the host machine fails and there is no external or backup storage, the application data may also be lost.

---

## Networking

- Docker uses the **bridge network** by default.
- The bridge network works only within a single Docker Host.
- It cannot provide communication between containers running on different Docker Hosts.
