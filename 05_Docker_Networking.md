# **Docker Networking**
Networking is a crucial aspect of Docker, as it controls how containers communicate with each other and the outside world. This guide will:
- Explain the different Docker networking modes
- Demonstrate how to connect containers inside Docker networks
- Provide real-world examples
- Analyze command outputs to ensure clarity

---

## **1. Understanding Docker Networks**
Docker provides multiple networking modes, each suited for different use cases.

| Network Mode | Description |
|-------------|-------------|
| **Bridge (Default)** | Containers are isolated but can communicate using explicit port mappings. |
| **Host** | The container shares the host's network stack, removing network isolation. |
| **None** | The container has no network access, making it useful for security. |
| **Overlay** | Used in multi-host Swarm setups for inter-container communication. |

### **How to List Available Networks**
Before diving into practical scenarios, it is important to check the existing Docker networks on your machine.

```sh
docker network ls
```

### **Expected Output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
7b369f1d82a3   bridge    bridge    local
44e59ebc37a7   host      host      local
4bb30e91a290   none      null      local
```
- **bridge**: Default network mode. Containers in this network can communicate using IP addresses.
- **host**: The container shares the host machine’s network.
- **none**: The container has no network access.

Now, let’s apply this knowledge in real-world scenarios.

---

## **Scenario 1: Running a Standalone Application**
### **Problem Statement**
A React application needs to be deployed using Docker and must be accessible from a web browser. Before running the application, we must ensure the necessary Docker image (Nginx) is available on the system.

### **Step 1: Check if the Nginx Image is Available Locally**
Before pulling an image from Docker Hub, check whether it already exists on the local system using:

```sh
docker images
```

### **Expected Output (If the Image is Already Available)**
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    6b361163bff3   3 days ago     142MB
```
- `REPOSITORY`: The name of the image.
- `TAG`: The image version (e.g., `latest`).
- `IMAGE ID`: A unique identifier for the image.
- `CREATED`: When the image was created.
- `SIZE`: The storage size of the image.

If the **Nginx image is not listed**, proceed to the next step.

### **Step 2: Search for the Nginx Image on Docker Hub**
To find available versions of Nginx on Docker Hub, use:

```sh
docker search nginx
```

### **Expected Output**
```
NAME                               DESCRIPTION                                      STARS   OFFICIAL   AUTOMATED
nginx                              Official Nginx Docker Image                      18542   [OK]       
jwilder/nginx-proxy                Automated Nginx reverse proxy                    2342
bitnami/nginx                      Bitnami Nginx container                          1234
```
- The **official** Nginx image is marked with `[OK]`.
- Other versions (e.g., `bitnami/nginx`) are available but are maintained by different organizations.

Since we want the **official** version, we can now **download it**.

### **Step 3: Pull the Nginx Image (If Not Available Locally)**
If the image is missing, download it from Docker Hub using:

```sh
docker pull nginx
```

### **Expected Output**
```
Using default tag: latest
latest: Pulling from library/nginx
6ec8c9369e08: Pull complete
d695473f6229: Pull complete
Digest: sha256:XXXXXXXXXXXX
Status: Downloaded newer image for nginx:latest
```
- The image layers are downloaded and stored locally.
- The `Status: Downloaded newer image` confirms the successful download.

### **Step 4: Check If Any Containers Are Running**
Before proceeding, check if any containers are already running:

```sh
docker ps
```

### **Expected Output (If No Containers Are Running)**
```
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```
If no containers are listed, no applications are currently running.

If any containers **are running**, stop them if necessary using:

```sh
docker stop <container_id>
```

### **Step 5: Run the Nginx Container**
```sh
docker run -d -p 8080:80 --name myweb nginx
```

### **Command Breakdown**
- `docker run`: Starts a new container.
- `-d`: Runs the container in detached mode (in the background).
- `-p 8080:80`: Maps port 8080 on the host machine to port 80 inside the container. This allows the application to be accessed via port 8080.
- `--name myweb`: Assigns the container a readable name for easy management.
- `nginx`: Specifies the Nginx image, which serves static files.

### **Step 6: Verify the Container is Running**
```sh
docker ps
```

### **Expected Output**
```
CONTAINER ID   IMAGE    COMMAND                  PORTS                    NAMES
c1d5f7e2a15b   nginx    "/docker-entrypoint.…"   0.0.0.0:8080->80/tcp     myweb
```
- The container is now running and accessible.

### **Step 7: Access the Application**
Open a web browser and go to:
```
http://localhost:8080
```
You should see the Nginx welcome page.

### **Key Takeaways**
- The `docker images` command helps verify if an image is available locally.
- The `docker search` command allows searching for images on Docker Hub.
- The `docker pull` command downloads an image when needed.
- Running `docker ps` ensures no conflicting containers are running before deployment.
- Port mapping enables external access to the containerized application.

---

## **Scenario 2: Two Containers Communicating (Microservices)**
### **Problem Statement**
A frontend application needs to fetch data from a backend API. Using `localhost` will not work inside Docker.

### **Solution**
A custom bridge network will be created so that containers can communicate using names instead of IP addresses.

### **Step 1: Create a Custom Network**
```sh
docker network create mynetwork
```

### **Command Breakdown**
- `docker network create mynetwork`: Creates a new bridge network named `mynetwork`. This allows multiple containers to communicate using names instead of IP addresses.

### **Step 2: Run the Backend API**
```sh
docker run -d --name backend --network mynetwork node:18
```

### **Command Breakdown**
- `docker run -d`: Runs the container in detached mode.
- `--name backend`: Assigns the container the name `backend`.
- `--network mynetwork`: Connects the container to `mynetwork`, allowing it to communicate with other containers in the same network.
- `node:18`: Uses the Node.js 18 image for the backend service.

### **Step 3: Run the Frontend**
```sh
docker run -d --name frontend --network mynetwork nginx
```

### **Step 4: Verify Network Connections**
```sh
docker network inspect mynetwork
```

### **Expected Output**
```
"Containers": {
    "backend": {
        "Name": "backend",
        "IPv4Address": "192.168.1.2/24"
    },
    "frontend": {
        "Name": "frontend",
        "IPv4Address": "192.168.1.3/24"
    }
}
```
- The `frontend` and `backend` containers now have internal IP addresses.
- They can communicate using names instead of IP addresses.

### **Step 5: Test Communication**
To confirm communication, execute the following command inside the frontend container:
```sh
docker exec -it frontend curl backend:3000
```
- `docker exec -it`: Runs a command inside a running container.
- `frontend`: Specifies the container where the command should be executed.
- `curl backend:3000`: Tries to access the backend API inside the same network.

If successful, this will return a response from the backend.

### **Key Takeaways**
- A custom network **eliminates the need for IP tracking**.
- Containers in the same network **can communicate by name**.
- This method is useful for **microservices architectures**.

---

## **Scenario 3: Connecting a Database to an Application**
### **Problem Statement**
A Flask API needs to connect to a PostgreSQL database inside Docker.

### **Solution**
A custom network will be created to connect the application and database containers.

### **Step 1: Create a Network**
```sh
docker network create dbnetwork
```

### **Step 2: Start PostgreSQL**
```sh
docker run -d --name database --network dbnetwork -e POSTGRES_PASSWORD=mysecretpassword postgres
```

### **Command Breakdown**
- `-e POSTGRES_PASSWORD=mysecretpassword`: Sets the database password.
- `--network dbnetwork`: Ensures the database is only accessible within the network.

### **Step 3: Start Flask API**
```sh
docker run -d --name flaskapp --network dbnetwork python:3.9
```

### **Step 4: Configure Flask**
Modify the Flask configuration:
```python
DATABASE_URL = "postgresql://postgres:mysecretpassword@database:5432/mydb"
```
- `database`: The PostgreSQL container name, which can now be used instead of an IP.

### **Key Takeaways**
- The database remains **hidden** from external access.
- The application can resolve `database` by name inside the network.

---
