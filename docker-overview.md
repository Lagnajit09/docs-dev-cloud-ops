# Docker 101: Your First Steps into Containerization

![](https://logos-world.net/wp-content/uploads/2021/02/Docker-Logo.png)




### Docker documentation: [DockerCompleteGuide.pdf](https://github.com/Lagnajit09/Docker-Learning/blob/master/docker-guide.pdf)

[Sample-Code-Files](https://github.com/Lagnajit09/Docker-Learning)

## What is Docker?

Docker is an open-source platform that enables developers to automate the deployment of applications inside lightweight, portable containers. These containers include everything needed to run an application, such as the code, runtime, libraries, and dependencies, ensuring that it runs consistently across different environments.

## Why Use Docker?

1. **Portability**: Docker containers can run on any system that supports Docker, whether it's your local machine, a cloud server, or a virtual machine.
2. **Consistency**: With Docker, you can ensure that your application behaves the same in development, testing, and production environments.
3. **Efficiency**: Docker containers are lightweight and share the host system's kernel, making them more efficient than traditional virtual machines.
4. **Isolation**: Each container runs in its own isolated environment, preventing conflicts between applications and ensuring better security.

## Key Concepts in Docker

1. **Images**: Immutable snapshots of an application and its environment. Images are used to create containers.
2. **Containers**: Running instances of images. Containers can be started, stopped, moved, and deleted.
3. **Dockerfile**: A script containing instructions on how to build a Docker image.
4. **Docker Hub**: A cloud-based registry where you can find and share Docker images.

## Installing Docker

Docker can be installed on various operating systems, including Windows, macOS, and Linux. Here's a brief overview of the installation process for each:

1. **Windows**:
    - Download and install Docker Desktop from the [official website](https://www.docker.com/).
    - Follow the installation instructions and restart your computer if prompted.
2. **macOS**:
    - Download and install Docker Desktop from the [official website](https://www.docker.com/).
    - Follow the installation instructions and grant the necessary permissions.
3. **Linux**:
    - Follow the installation instructions for your specific Linux distribution on the [Docker website](https://www.docker.com/).
    
    Verify the installation by running `docker --version` in the terminal.
    

## Your First Docker Container

Let's create and run your first Docker container. We'll use the official "hello-world" image to verify that Docker is working correctly.

1. Open your terminal or command prompt.
2. Run the following command:
    
    ```bash
    docker run hello-world
    ```
    
3. Docker will download the "hello-world" image and run it in a new container. You should see a message indicating that Docker is installed correctly.

## Build Your First Docker Image

### Creating a Docker Image

When you create a Docker image, you are packaging an application along with its dependencies and configuration settings into a single, self-contained unit. This image can then be used to create containers, which are instances of the application running in isolated environments.

### Example: Preparing a Meal Kit

Think of creating a Docker image as preparing a meal kit. You gather all the ingredients (code, libraries, dependencies) and instructions (configuration files) needed to make a meal (run the application). You then pack everything into a box (the Docker image) that can be shipped anywhere.

### Steps to Create a Docker Image

1. **Write a Dockerfile:**
    - A Dockerfile is a script that contains instructions on how to build the Docker image.
    
    Example:
    
    ```
    # Use an official Python runtime as a parent image
    FROM python:3.8-slim
    
    # Set the working directory in the container
    WORKDIR /app
    
    # Copy the current directory contents into the container at /app
    COPY . /app
    
    # Install any needed packages specified in requirements.txt
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Make port 80 available to the world outside this container
    EXPOSE 80
    
    # Define environment variable
    ENV NAME World
    
    # Run app.py when the container launches
    CMD ["python", "app.py"]
    ```
    
2. **Build the Image:**
    - Use the Docker CLI to build the image from the Dockerfile.
    
    ```bash
    docker build -t my-flask-app .
    ```
    

### Creating a Docker Container

When you create a Docker container, you are launching an instance of a Docker image. The container runs the application defined by the image in an isolated environment, which ensures consistency across different environments.

### Example: Cooking the Meal

Creating a Docker container is like cooking the meal using the meal kit. You take the box (Docker image), follow the instructions, and use the ingredients to prepare the meal (run the application). The meal (application) will taste the same no matter where you cook it (deploy it).

### Steps to Create a Docker Container

1. **Run the Container:**
    - Use the Docker CLI to create and start a container from the image.
    
    ```bash
    docker run -p 4000:80 my-flask-app
    ```
    

### Explanation:

1. **Docker Image Creation:**
    - **Gather Ingredients (Dependencies and Code):** The `FROM` instruction specifies the base image (Python runtime). The `COPY` instruction copies the application code and dependencies into the image. The `RUN` instruction installs the necessary dependencies.
    - **Pack Everything (Build the Image):** The `docker build` command packages the application, its dependencies, and configuration into the `my-flask-app` image.
2. **Docker Container Creation:**
    - **Unpack and Cook (Run the Container):** The `docker run` command creates a container from the `my-flask-app` image, exposing the application on port 4000 of the host machine. The `CMD` instruction in the Dockerfile specifies the command to run when the container starts, which in this case is to launch the Flask application.

By using Docker images and containers, developers can ensure that applications run consistently across different environments, similar to how a meal kit ensures a meal tastes the same regardless of where it is cooked.

## Docker Compose

### What is Docker Compose?

Docker Compose is a tool for defining and running multi-container Docker applications. With Docker Compose, you use a YAML file to configure your application's services, networks, and volumes. Then, with a single command, you can create and start all the services from your configuration.

### When to Use Docker Compose?

You should use Docker Compose when:

- You have a multi-container application that needs to be managed and orchestrated easily.
- You need to set up an application stack with multiple services (e.g., a web server, a database, a cache).
- You want to manage environments (development, testing, production) with different configurations.

### What Does `docker compose up` Do?

The `docker compose up` command does the following:

1. **Reads the  `compose.yaml` File:** Parses the configuration defined in the YAML file.
2. **Builds Images:** Builds the Docker images if they do not already exist.
3. **Creates and Starts Containers:** Creates and starts the containers as defined in the configuration.
4. **Sets Up Networking:** Configures the networking between containers as defined.
5. **Manages Volumes:** Sets up any volumes defined in the configuration.

### Example `compose.yaml`

Here's an example of a `compose.yaml` file for a simple web application consisting of a Node.js backend, a React frontend, and a MongoDB database.

```yaml
version: '3.8'

services:
  frontend:
    build: ./react-app
    ports:
      - "3000:80"
    depends_on:
      - backend

  backend:
    build: ./nodejs-app
    ports:
      - "5000:3000"
    environment:
      - MONGO_URL=mongodb://db:27017/mydatabase
    depends_on:
      - db

  db:
    image: mongo:4.2
    ports:
      - "27017:27017"
    volumes:
      - db-data:/data/db

volumes:
  db-data:
```

### Explanation:

1. **Version:**
    - Specifies the version of the Docker Compose file format. Here, it's `3.8`.
2. **Services:**
    - Defines the different services that make up the application.
    
    **Frontend Service:**
    
    - `build: ./react-app`: Specifies the build context for the React application, which is in the `./react-app` directory.
    - `ports: - "3000:80"`: Maps port 80 of the container to port 3000 on the host.
    - `depends_on: - backend`: Specifies that the frontend service depends on the backend service and will start after it.
    
    **Backend Service:**
    
    - `build: ./nodejs-app`: Specifies the build context for the Node.js application, which is in the `./nodejs-app` directory.
    - `ports: - "5000:3000"`: Maps port 3000 of the container to port 5000 on the host.
    - `environment: - MONGO_URL=mongodb://db:27017/mydatabase`: Sets the environment variable for the MongoDB connection URL.
    - `depends_on: - db`: Specifies that the backend service depends on the db service and will start after it.
    
    **DB Service:**
    
    - `image: mongo:4.2`: Specifies the MongoDB image to use.
    - `ports: - "27017:27017"`: Maps port 27017 of the container to port 27017 on the host.
    - `volumes: - db-data:/data/db`: Mounts the `db-data` volume to the `/data/db` directory in the container to persist data.
3. **Volumes:**
    - Defines the named volumes used by the services. Here, `db-data` is a named volume used by the MongoDB service to persist data.

### How to Use `docker compose up`

1. **Ensure Docker and Docker Compose are Installed:**
    - Install Docker and Docker Compose on your machine if not already installed.
2. **Create a `compose.yaml` File:**
    - Create a `compose.yaml` file in your project directory with the necessary configuration.
3. **Run the Command:**
    - Navigate to your project directory in the terminal and run the following command:
        
        ```bash
        docker compose up
        ```
        
    - This command will build the images (if not already built), create and start the containers, set up the networking, and manage volumes as defined in the `compose.yaml` file.

## Docker Compose with File Watching

### What is Docker Compose with Watch?

Docker Compose with watch allows you to automatically restart services when files change. This is particularly useful during development, as it enables hot-reloading of services, reducing the need to manually restart containers after code changes.

### When to Use Docker Compose with Watch?

You should use Docker Compose with watch when:

- You are developing applications and want to see changes reflected immediately without manually restarting containers.
- You need to ensure that services automatically restart when their source code or configuration files are modified.

### What Does `docker compose up` Do with Watch?

When used with file watching, `docker compose up` will:

1. **Watch for File Changes:** Monitor specified files or directories for changes.
2. **Restart Services:** Automatically restart the affected services when changes are detected.
3. **Streamline Development Workflow:** Enable a smoother and more efficient development process with immediate feedback on changes.
4. Run the command:
    
    ```bash
    docker compose watch
    ```
    

## Docker Scout

Docker Scout is a tool provided by Docker that helps developers identify and address security vulnerabilities and compliance issues in their Docker images. It performs deep scans of Docker images, examining their contents to detect known vulnerabilities in the software components and dependencies they include. Docker Scout provides detailed reports and actionable insights, allowing developers to improve the security and compliance of their containerized applications.