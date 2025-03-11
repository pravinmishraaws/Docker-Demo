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
docker run -d -p 80:80 -v ~/nginx-logs:/var/log/nginx --name myweb nginx
```

### **Command Breakdown**
- `docker run`: Starts a **new container**.
- `-d`: Runs the container in **detached mode** (in the background).
- `-p 80:80`: Maps **port 80** on the **host machine** to **port 80** inside the container.
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
c1d5f7e2a15b   nginx    "/docker-entrypoint.…"   0.0.0.0:80->80/tcp     myweb
```
The container is now **running and accessible**.


## **Step 8: Access the Application**
Open a **web browser** and go to:

```
http://PublicIP:8080
```

You should see the **Nginx welcome page**.


## **Step 9: Verify That Logs Are Being Written**
Check if Nginx logs are being saved on the host:

```sh
ls ~/nginx-logs
```

### **Expected Output**
```
access.log  error.log
```
This confirms that **logs are stored on the host machine**.

## **Step 10: Remove the Container and Check Log Persistence**
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


## **Key Takeaways**
- The `docker images` command **verifies if an image is available locally**.
- The `docker search` command **finds images on Docker Hub**.
- The `docker pull` command **downloads an image when needed**.
- The `docker ps` command **ensures no conflicting containers are running** before deployment.
- **Using a bind mount (`-v ~/nginx-logs:/var/log/nginx`) allows logs to persist even if the container is removed**.
- **Port mapping (`-p 80:80`) enables external access to the containerized application**.

---

# **Scenario 2: Two Containers Communicating (Microservices) with Persistent Data**
## **Problem Statement**
A **frontend application** needs to fetch data from a **backend API** inside Docker. Since each container runs in an **isolated environment**, using `localhost` will not work.

Additionally, we want to **persist data generated by the backend** so the frontend can retrieve it even if the backend container restarts.

To achieve this, we will:  
- **Use a custom bridge network** for inter-container communication.  
- **Use a shared Docker volume** to store and retrieve data between the backend and frontend.  

---

## **Step 1: Cleanup Previous Containers and Images**
Before setting up the new scenario, stop and remove any containers and images from the previous scenario.

### **Stop and Remove the Running Container**
Check if any containers are running:

```sh
docker ps
```

### **Expected Output (If Any Containers Are Running)**
```
CONTAINER ID   IMAGE    COMMAND                  PORTS                    NAMES
c1d5f7e2a15b   nginx    "/docker-entrypoint.…"   0.0.0.0:80->80/tcp       myweb
```
If the `myweb` container is running, stop it:

```sh
docker stop myweb
```

Remove the container:

```sh
docker rm myweb
```

### **Remove Unused Images**
Check for unused images:

```sh
docker images
```

If the **Nginx image** is no longer needed, remove it:

```sh
docker rmi nginx
```

At this point, the system is **clean** and ready for the new scenario.

---

## **Step 2: Set Up the Folder Structure**
Before writing any Dockerfiles, create the project structure:

```sh
mkdir -p two-tier-app/{frontend,backend}
```

### **Folder Structure**
```
two-tier-app/
│-- frontend/
│   └── Dockerfile
│-- backend/
│   └── Dockerfile
```

Navigate into the backend folder and create an empty Dockerfile:

```sh
cd two-tier-app/backend
touch Dockerfile
```

Repeat for the frontend:

```sh
cd ../frontend
touch Dockerfile
```

---

## **Step 3: Create a Custom Docker Network**
Before running any containers, create a **custom network** to allow communication:

```sh
docker network create mynetwork
```

### **Verify the Network**
```sh
docker network ls
```

### **Expected Output**
```
NETWORK ID     NAME        DRIVER    SCOPE
a9f58b94c8a0   mynetwork   bridge    local
```
This confirms that `mynetwork` was created.

---

## **Step 4: Create a Shared Volume for Data Persistence**
Create a **Docker volume** for shared storage:

```sh
docker volume create shared-data
```

### **Verify the Volume**
```sh
docker volume ls
```

### **Expected Output**
```
DRIVER    VOLUME NAME
local     shared-data
```
This volume allows **backend to write data** and **frontend to read it**.

---

## **Step 5: Build and Run the Backend Service**
### **A: Create the Backend Dockerfile**
Inside `two-tier-app/backend/Dockerfile`, add:

```Dockerfile
# Use the official Node.js image
FROM node:18

# Set the working directory inside the container
WORKDIR /app

# Install required dependencies
RUN npm init -y && npm install express fs

# Create index.js with API logic
RUN echo "const express = require('express'); \
const fs = require('fs'); \
const app = express(); \
const DATA_FILE = '/data/message.txt'; \
app.get('/write', (req, res) => { fs.writeFileSync(DATA_FILE, 'Hello from Backend!'); res.send('Data written!'); }); \
app.listen(80, () => console.log('Backend running on port 80'));" > index.js

# Expose port 80 to the host
EXPOSE 80

# Run the application
CMD ["node", "index.js"]
```

### **B: Build and Run the Backend Container**
Navigate to the `backend` directory and build the image:

```sh
cd two-tier-app/backend
docker build -t backend-app .
```

Now, run the **backend container** with the shared volume:

```sh
docker run -d --name backend --network mynetwork -v shared-data:/data backend-app
```

### **Verify the Backend is Running**
```sh
docker ps
```

### **Expected Output**
```
CONTAINER ID   IMAGE         COMMAND                  PORTS                   NAMES
a8d2c9f8e6a7   backend-app   "node index.js"         80/tcp                   backend
```

The **backend is running and writing data to the shared volume (`/data/message.txt`)**.

---

## **Step 6: Build and Run the Frontend Service**
### **A: Create the Frontend Dockerfile**
Inside `two-tier-app/frontend/Dockerfile`, add:

```Dockerfile
# Use the official Node.js image
FROM node:18

# Set the working directory inside the container
WORKDIR /app

# Install required dependencies
RUN npm init -y && npm install express fs

# Create index.js to serve data
RUN echo "const express = require('express'); \
const fs = require('fs'); \
const app = express(); \
const DATA_FILE = '/data/message.txt'; \
app.get('/', (req, res) => { res.send(fs.existsSync(DATA_FILE) ? fs.readFileSync(DATA_FILE, 'utf8') : 'No data found.'); }); \
app.listen(80, () => console.log('Frontend running on port 80'));" > index.js

# Expose port 80 to the host
EXPOSE 80

# Run the application
CMD ["node", "index.js"]
```

### **B: Build and Run the Frontend Container**
Navigate to the `frontend` directory and build the image:

```sh
cd ../frontend
docker build -t frontend-app .
```

Now, run the **frontend container** with the shared volume:

```sh
docker run -d --name frontend --network mynetwork -v shared-data:/data -p 80:80 frontend-app
```

### **Verify the Frontend is Running**
```sh
docker ps
```

### **Expected Output**
```
CONTAINER ID   IMAGE         COMMAND                  PORTS                   NAMES
b1d5a7c8e9f2   frontend-app  "node index.js"         0.0.0.0:80->80/tcp       frontend
```

---

## **Step 7: Test Data Sharing Between Backend and Frontend**

Now, let's verify that data is written by the backend and read by the frontend while ensuring persistence across multiple writes.  

---

### **Step A: Write Data from the Backend**  
1. Run the following command to write data from the backend:  
   ```sh
   docker exec backend curl http://localhost/write
   ```
2. **Expected Output:**  
   ```
   Data written!
   ```
   This confirms that the backend has written "Hello from Backend!" into the shared volume.  

---

### **Step B: Read Data from the Frontend**  
1. Run the following command on your host machine to fetch the stored data via the frontend:  
   ```sh
   http://<PublicIP>
   ```
2. **Expected Output:**  
   ```
   Hello from Backend!
   ```
   This confirms that the frontend is reading the data written by the backend.  

---

### **Step C: Manually Modify Data Multiple Times**  
To further test data persistence, let's manually write new data and verify it updates correctly.  

1. Write a custom message inside the backend container:  
   ```sh
   docker exec backend sh -c "echo 'Test Data 1' > /data/message.txt"
   ```
2. Read the updated data via the frontend:  
   ```sh
   http://<PublicIP>
   ```
   **Expected Output:**  
   ```
   Test Data 1
   ```
3. Write another update to simulate new data being generated:  
   ```sh
   docker exec backend sh -c "echo 'Test Data 2 - New Update' > /data/message.txt"
   ```
4. Read the new data from the frontend:  
   ```sh
   http://<PublicIP>
   ```
   **Expected Output:**  
   ```
   Test Data 2 - New Update
   ```
   This confirms that every update to the backend's data file is immediately reflected in the frontend.  

---


### **Step D: Manually Verify Data Inside the Shared Volume**  
We can directly inspect the stored data in `shared-data` to confirm persistence.  

1. List all Docker volumes:  
   ```sh
   docker volume ls
   ```
   **Expected Output:**  
   ```
   DRIVER    VOLUME NAME
   local     shared-data
   ```
2. Inspect the volume’s data manually:  
   ```sh
   docker run --rm -v shared-data:/data alpine cat /data/message.txt
   ```
   **Expected Output:**  
   ```
   Test Data 2 - New Update
   ```
   This confirms that the data is stored persistently within the volume, even if the backend container is restarted.  

---

### **Step E: Restart Backend and Verify Persistence**  
Since Docker volumes persist even if containers are removed, let's restart the backend and check if the data remains.  

1. Stop and remove the backend container:  
   ```sh
   docker stop backend
   docker rm backend
   ```
2. Re-run the backend container:  
   ```sh
   docker run -d --name backend --network mynetwork -v shared-data:/data backend-app
   ```
3. Read data from the frontend (it should still be available):  
   ```sh
   http://<PublicIP>
   ```
   **Expected Output:**  
   ```
   Test Data 2 - New Update
   ```
   This confirms that data remains stored in the volume even after the backend is restarted.  



# **Where is the `shared-data` volume located?**
The **Docker volume** `shared-data` is **not a regular directory** on your host machine. Instead, Docker manages it internally and stores it under **Docker's volume storage location**.

To find it, you need to use **Docker commands** instead of regular `cd` and `ls`.

---

## **How to Locate the `shared-data` Volume on the Host Machine**
### **1. Find the Volume Path**
Run:
```sh
docker volume inspect shared-data
```

### **Expected Output**
```json
[
    {
        "CreatedAt": "2025-03-11T12:34:56Z",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/shared-data/_data",
        "Name": "shared-data",
        "Scope": "local"
    }
]
```
- The **`Mountpoint`** field shows where the volume is stored on the host machine.
- In this example, the volume is located at:
  ```
  /var/lib/docker/volumes/shared-data/_data
  ```

---

### **2. List Files Inside the Volume (From the Host)**
Now, you can check the files **directly from the host machine** using:
```sh
ls -l /var/lib/docker/volumes/shared-data/_data
```

If `message.txt` exists, you should see:
```
-rw-r--r--  1 root root  20 Mar 11 12:45 message.txt
```

---

### **3. Try `cd` Inside the Volume (From the Host)**
You can navigate into the volume directory:
```sh
cd /var/lib/docker/volumes/shared-data/_data
ls -l
```
If the file `message.txt` exists, it will be listed.

---

## **Why Can’t You See It with `cd` Normally?**
Docker **does not mount volumes in a user-visible directory by default**. Volumes are stored inside `/var/lib/docker/volumes/` but are **managed by Docker**.

- Unlike **bind mounts**, which are mapped to an existing directory on the host, **volumes are managed by Docker internally**.
- This is why you **cannot see the volume** with a simple `cd ~/shared-data` unless you explicitly create a bind mount.

---

## **Alternative: Check Volume Files from Inside a Running Container**
Instead of checking from the host, you can **inspect the volume from a running container**:

1. Open a shell inside any container using the volume:
```sh
docker exec -it backend sh
```
2. Navigate to the mounted volume directory:
```sh
cd /data
ls -l
```
3. Read the file:
```sh
cat message.txt
```

---

## **Key Takeaways**
- **Docker volumes are stored under `/var/lib/docker/volumes/`** but are not directly accessible using `cd` unless you navigate inside Docker’s storage directory.
- **You can locate them using** `docker volume inspect shared-data`.
- **To see the volume contents from the host, use `ls` inside `/var/lib/docker/volumes/shared-data/_data/`.**
- **To check the volume from inside a running container, use `docker exec -it backend sh` and navigate to `/data`.**  


# **How to Make Data Truly Persistent with Azure Managed Disks**  
Currently, **Docker volumes store data inside the VM**, but if the **VM is deleted or restarted, all stored data is lost**. To ensure **true persistence**, we need to store the data on **external storage**. One of the best options in Azure is **Azure Managed Disks**.  

---

## **Option 1: Use an Azure Managed Disk for Persistent Storage**  
Azure Managed Disks allow us to **attach an external disk** to the VM, ensuring that data persists even if the VM is stopped, restarted, or deleted.  

---

### **1. Create and Attach an Azure Managed Disk**  
#### **A: Create a Managed Disk in Azure**  
1. Open the **Azure Portal**.  
2. Go to **Disks** → **Create a new Managed Disk**.  
3. Choose:  
   - **Disk Type**: Standard HDD, Standard SSD, or Premium SSD.  
   - **Size**: Choose a size that meets your storage needs.  
   - **Resource Group**: Select the resource group for your VM.  
   - **Encryption**: Use default encryption unless a specific policy is required.  
4. Click **Create**.  

#### **B: Attach the Disk to Your VM**  
1. Open **Azure Portal**.  
2. Go to **Virtual Machines** → Select your VM.  
3. Click **Disks** → **Attach existing disk**.  
4. Select the **Managed Disk** created earlier.  
5. Click **Save**.  

---

### **2. Mount the Disk on the VM**  
Once the disk is attached to the VM, it needs to be **formatted and mounted**.  

#### **A: List Available Disks**  
Run the following command to find the **newly attached disk**:  
```sh
lsblk
```  
You should see an unmounted disk, such as `/dev/sdc` or `/dev/sdb`.  

#### **B: Create a Filesystem on the New Disk**  
Replace `/dev/sdc` with your actual disk name:  
```sh
sudo mkfs -t ext4 /dev/sdc
```  

#### **C: Create a Mount Directory**  
```sh
sudo mkdir -p /mnt/data-storage
```  

#### **D: Mount the Disk**  
```sh
sudo mount /dev/sdc /mnt/data-storage
```  

#### **E: Make the Mount Persistent After Reboots**  
To ensure the disk remains mounted even after a VM restart, update `/etc/fstab`:  
```sh
echo '/dev/sdc /mnt/data-storage ext4 defaults,nofail 0 2' | sudo tee -a /etc/fstab
```  

---

### **3. Use the Mounted Disk as Persistent Storage for Docker**  
Now, update **Docker to use the Azure Managed Disk as a persistent storage location**.  

#### **A: Run the Backend Container with a Bind Mount to the Managed Disk**  
```sh
docker run -d --name backend --network mynetwork -v /mnt/data-storage:/data backend-app
```  
- This ensures that all backend data is stored in `/mnt/data-storage`, which is on the Azure Managed Disk.  
- If the VM restarts or is recreated, the data will still be available.  

#### **B: Verify Data Persistence**  
1. **Write Data from the Backend:**  
   ```sh
   docker exec backend curl http://localhost/write
   ```  
2. **Manually Check if the File Exists on the Mounted Disk:**  
   ```sh
   ls -l /mnt/data-storage
   ```  
   **Expected Output:**  
   ```
   -rw-r--r--  1 root root  20 Mar 11 12:45 message.txt
   ```  
3. **Reboot the VM and Verify the Data Again:**  
   ```sh
   curl http://localhost
   ```  
   The response should still be:  
   ```
   Hello from Backend!
   ```  

---

## **Key Takeaways**  
- **Azure Managed Disks ensure data persistence across VM restarts and deletions.**  
- **The disk is mounted at `/mnt/data-storage`**, and Docker stores data there using a bind mount.  
- **Even if the VM is deleted, the disk can be reattached to a new VM, ensuring no data loss.**  
- **This method is better than Docker volumes stored in `/var/lib/docker/volumes/`, which would be lost if the VM is deleted.**  

## Option 2: Use an NFS Storage
## Option 3: Use Cloud Storage Volumes for Docker
## Option 4: Store Data in a Database Instead



