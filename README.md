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

docker best practices:
==========================
minimum and offical images
multi stage builds
optimising layers and combining RUN commands
non root containers
use customized networks
implementing volumes
COPY over ADD
docker igonre not load everything into docker
impelementing health cheks
limiting resources
getting secreat from secreat manager
implementing volumes

cd catalogue/
docker build -t lakshmi1092//catalogue:v1
docker login -u lakshmi1092
docker push lakshmi1092/catalogue:v1
docker compose up -d
docker ps
docker images

for i in cart catalogue user ; do cd $i; docker build -t lakshmi1092//$i:v1 . ; docker push lakshmi/$i:v1 ;cd ..;   done
docker images
docker compose up -d
docker restart frontend


source code ---> complie ----> bytecode(intermediate language).
for develping this source code we need jdk
JDK ---> java development kit
jdk ---> no need of developement env(runs bytecode)
in java we did everything using maven

Docker Architecutre:
===================
client ---> docker CLI where we can run our docker commands
host ----> where docker is running, docker deamon(continousy running)
repos ---> local and central repo

what happen when we run 
docker run nginx
----------------
1. 1st it checks the image is in local or not
2. if exstis then it will create the conatiner
3. if not exsit then it wil pull from registry and  create the conatiner and send the o/p to client
4. they are docker volumes and networkig we can configure 

Disadvanatges
==============
auto scaling: There is no deafult auto scaling methods
load blancing: no load blancing components to blance the traffic b/w the containers
reliablity: If container crashes it will not automatically restart(no self healing)
what if docker host crash: all conatiner are goes down
what about storage: if docker host crashes we loose data also beacuse docker is manging volumes on the same host
networking is in bridge mode, if you have multiple docker hosts bridge host will not work
