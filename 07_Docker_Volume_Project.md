# Practical Demonstrations: Integrating Docker Volumes

# **Scenario 1: Running a Standalone Application with Persistent Logs**

## **Problem Statement**
A React application needs to be deployed using **Docker** and must be accessible from a **web browser**. However, there is an issue: if the container stops or is deleted, **all logs are lost**.

To solve this problem, we will use a **bind mount** to store Nginx logs on the **host machine**, ensuring that logs remain available even if the container is removed.

## **Step 1: Check if the Nginx Image is Available Locally**
Before pulling an image from Docker Hub, check whether it already exists on the local system using:

```sh
docker images
```

### **Expected Output (If the Image is Already Available)**
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    6b361163bff3   3 days ago     142MB
```

- **REPOSITORY**: The name of the image.
- **TAG**: The image version (e.g., `latest`).
- **IMAGE ID**: A unique identifier for the image.
- **CREATED**: When the image was created.
- **SIZE**: The storage size of the image.

If the Nginx image is **not listed**, proceed to the next step.

## **Step 2: Search for the Nginx Image on Docker Hub**
To find available versions of Nginx on **Docker Hub**, use:

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
- The **official Nginx image** is marked with `[OK]`.
- Other versions (e.g., `bitnami/nginx`) are available but maintained by different organizations.

Since we want the official version, we can now download it.

## **Step 3: Pull the Nginx Image (If Not Available Locally)**
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
The **image layers** are downloaded and stored locally.  
The **`Status: Downloaded newer image`** confirms the successful download.


## **Step 4: Check If Any Containers Are Running**
Before proceeding, check if any containers are already running:

```sh
docker ps
```

### **Expected Output (If No Containers Are Running)**
```
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```
If no containers are listed, no applications are currently running.

If any containers are running, stop them if necessary using:

```sh
docker stop <container_id>
docker rm <container_id>
```

## **Step 5: Create a Logs Directory on the Host**
To ensure logs persist, create a **directory on the host**:

```sh
mkdir -p ~/nginx-logs
```

This directory will store **Nginx logs** outside the container.


## **Step 6: Run the Nginx Container with Persistent Logs**
Run the Nginx container while **mounting the logs directory**:

```sh
docker run -d -p 8080:80 -v ~/nginx-logs:/var/log/nginx --name myweb nginx
```

### **Command Breakdown**
- `docker run`: Starts a **new container**.
- `-d`: Runs the container in **detached mode** (in the background).
- `-p 8080:80`: Maps **port 8080** on the **host machine** to **port 80** inside the container.
- `-v ~/nginx-logs:/var/log/nginx`:  
  - **Maps the Nginx logs directory** (`/var/log/nginx`) inside the container to **`~/nginx-logs`** on the host.
  - **Ensures logs persist** even if the container is stopped or deleted.
- `--name myweb`: Assigns the container a **readable name** (`myweb`).
- `nginx`: Specifies the **Nginx image**, which serves **static files**.

## **Step 7: Verify the Container is Running**
Check if the container is running:

```sh
docker ps
```

### **Expected Output**
```
CONTAINER ID   IMAGE    COMMAND                  PORTS                    NAMES
c1d5f7e2a15b   nginx    "/docker-entrypoint.…"   0.0.0.0:8080->80/tcp     myweb
```
The container is now **running and accessible**.

## **Step 8: Verify That Logs Are Being Written**
Check if Nginx logs are being saved on the host:

```sh
ls ~/nginx-logs
```

### **Expected Output**
```
access.log  error.log
```
This confirms that **logs are stored on the host machine**.

## **Step 9: Remove the Container and Check Log Persistence**
Stop and remove the container:

```sh
docker stop myweb
docker rm myweb
```

Verify that the logs are still present:

```sh
ls ~/nginx-logs
```

### **Expected Output (Logs Persist)**
```
access.log  error.log
```
This confirms that **even after the container is removed, logs remain on the host machine**.

## **Step 10: Access the Application**
Open a **web browser** and go to:

```
http://PublicIP:8080
```

You should see the **Nginx welcome page**.

## **Key Takeaways**
- The `docker images` command **verifies if an image is available locally**.
- The `docker search` command **finds images on Docker Hub**.
- The `docker pull` command **downloads an image when needed**.
- The `docker ps` command **ensures no conflicting containers are running** before deployment.
- **Using a bind mount (`-v ~/nginx-logs:/var/log/nginx`) allows logs to persist even if the container is removed**.
- **Port mapping (`-p 8080:80`) enables external access to the containerized application**.

---

