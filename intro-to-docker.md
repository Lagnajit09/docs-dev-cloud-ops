# 📦 **Introduction to Docker**

![](https://logos-world.net/wp-content/uploads/2021/02/Docker-Logo.png)

## 🚀 Introduction

### What is Docker?
**Docker** is a **containerization platform** that allows you to create, deploy, and manage lightweight, portable containers.


## 🛠️ Understanding Containers

### What are Containers?
- Containers are **isolated execution environments** that virtualize at the **operating system level**.
- Containers package applications with their dependencies (libraries, files) to ensure they "work anywhere."

![containers-vs-traditional](https://github.com/Lagnajit09/docs-dev-cloud-ops/blob/main/assets/comparison-docker.png?raw=true)

### Why Containers?
![usecase-1](https://github.com/Lagnajit09/docs-dev-cloud-ops/blob/main/assets/docker-use1.png?raw=true)
- Without adding any special functionality, container platforms (like Docker) can help us solve an age-old technological problem: the one where it "works on my machine" but not anywhere else. If an application is containerized, then an image of the very containers used in production can be supplied to developers and administrators to launch on their own workstations. This way it works on all machines.

- Similarly, when migrating applications or services, it may be beneficial to containerize the existing infrastructure first. That way, should we need to redeploy or migrate again in the future, we can simply redeploy the container instead of having to set up the entire environment. Even in instances where configuration management is used, deploy times might be sped up by switching to a container-based architecture.

![usecase-2](https://github.com/Lagnajit09/docs-dev-cloud-ops/blob/main/assets/docker-use2.png?raw=true)

- Containers can also affect how we architect our applications. Instead of monolithic applications that store all services as part of one framework, we can instead use microservices to isolate each individual parts of an application. When paired with orchestration, this can be especially powerful.

- Containers are used alongside an orchestration platform. Docker, specifically, tends to be paired with Kubernetes or Docker Swarm. Once paired with orchestration (and generally some kind of monitoring to report changes) a container platform can enhance our reliability and response times to issues and outages.

- This means we can watch for the CPU load on our containers, and when a certain limit is reached, automatically deploy or remove containers as needed to match the changes in load. This allows for an elastic architecture.

- Similarly, we can monitor for problems and then automatically terminate and replace any failing containers, creating a self-healing architecture. Even in instances where the container's build itself is failing, containers have their benefits. For example, it can often be quicker to remove and replace a number of containers than it is to redeploy to virtual machines.

- Containers also fit nicely into the concept of continuous delivery and continuous integration as a whole, the DevOps idea of deploying often and automatically based on changes made to some master repository. This means when we make changes to an application and push those changes to its production branch, a continuous integration platform will generate a new container image build, test the build, and deploy that build (assuming it passed all tests) by removing all old containers and replacing them with the new version.


## 🔧 Setting Up Docker

### Docker Installation (on Ubuntu)
Run the following commands:

1. Update system:
    ```bash
    sudo apt update
    sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    ```

2. Add Docker's GPG key:
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

3. Add Docker repository:
    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    sudo apt update
    ```

4. Install Docker:
    ```bash
    sudo apt install docker-ce docker-ce-cli containerd.io
    ```

5. Add your user to Docker group to avoid using `sudo`:
    ```bash
    sudo usermod -aG docker $USER
    ```

6. Verify installation:
    ```bash
    docker run hello-world
    ```

### What Happens During `docker run hello-world`?
![](https://github.com/Lagnajit09/docs-dev-cloud-ops/blob/main/assets/helloworld-docker.png?raw=true)
1. Docker client sends a task to the daemon.
2. Docker daemon looks for the `hello-world` image locally.
3. If not found, it pulls the image from **Docker Hub**.
4. A container is created and runs the image.


## 🧩 Managing Containers

### Launching Containers

1. Start a container:
    ```bash
    docker run -it --name demo-container alpine
    ```

2. List running containers:
    ```bash
    docker container ls
    ```

3. List all containers (including stopped):
    ```bash
    docker container ls -a
    ```

### Persistent Containers
Run a container in the background:
```bash
docker run -dt --name my-container --restart always alpine
```


## 📂 Working with Images

### What is a Docker Image?
A **Docker Image** is a lightweight, stand-alone, executable package that contains:
- Code
- Runtime
- Libraries
- Environment variables

---

### **Creating Your First Dockerfile**

A **Dockerfile** is a blueprint for building Docker images. Example for a Node.js app:

**Dockerfile:**
```Dockerfile
FROM node:10-alpine
WORKDIR /home/node/app
COPY package*.json ./
RUN npm install
COPY . .
USER node
EXPOSE 8080
CMD [ "node", "index.js" ]
```

### Build and Run
1. Build the image:
    ```bash
    docker build . -t my-node-app
    ```

2. Run the container:
    ```bash
    docker run -dt -p 8080:8080 --name node-container my-node-app
    ```

---

## 🌍 Docker Hub and Sharing Images

### Pushing to Docker Hub
1. Log in to Docker Hub:
    ```bash
    docker login --username=<your-username>
    ```

2. Tag your image:
    ```bash
    docker tag my-node-app <your-username>/my-node-app:latest
    ```

3. Push the image:
    ```bash
    docker push <your-username>/my-node-app
    ```

---

## 🧰 Advanced Docker Commands

### Copy Files to/from a Container
```bash
docker cp localfile.txt container:/path/to/destination/
docker cp container:/path/from/source/ localfile.txt
```

### Inspect a Container's details
```bash
docker inspect container-name 
```

### Resource Monitoring
```bash
docker stats
```

### Clean Up Stopped Containers
```bash
docker container prune
```

---

## 🐳 Difference Between Containers and Images

| **Aspect**            | **Docker Image**                                           | **Docker Container**                                       |
|------------------------|----------------------------------------------------------|----------------------------------------------------------|
| **Definition**         | A Docker Image is a **blueprint** or template used to create containers. | A Docker Container is a **running instance** of a Docker image. |
| **State**              | **Static** - Cannot be modified once built.              | **Dynamic** - Can be started, stopped, or modified at runtime. |
| **Purpose**            | Acts as the **template** that contains all dependencies, code, and environment setup. | The **runtime environment** where applications actually execute. |
| **Storage**            | Stored as **layers** in a read-only format.              | Runs as a **read-write layer** created from the image.   |
| **Creation**           | Built using a **Dockerfile** and `docker build` command. | Created using `docker run` from an image.               |
| **Examples**           | `ubuntu:latest`, `node:10-alpine`, `hello-world`          | Running `docker run ubuntu` starts a container instance. |
| **Persistence**        | Permanent. An image remains unchanged until rebuilt.     | Temporary. Containers can be started, stopped, and removed. |
| **Command Examples**   | - List images: `docker image ls` <br> - Remove image: `docker image rm` | - Run container: `docker run <image>` <br> - List containers: `docker container ls` |

---

### Analogy
- **Image:** Like a **recipe** or template to bake a cake.
- **Container:** The **actual baked cake** you eat, based on the recipe.



## 🚀 What's Next?
Congratulations! You've learned how to:
- Install Docker
- Build and manage containers
- Create and share images
- Work with Dockerfiles

Next steps: Explore orchestration tools like **Kubernetes** and container security best practices.

---

This is the **ultimate guide** for beginners ready to learn Docker from scratch! 📦