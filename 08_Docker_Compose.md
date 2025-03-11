## **Docker Compose for Multi-Tier Architecture (Frontend, Backend, Database)**

### **Problem Statement**
Previously, we set up a **multi-tier web application** using **Docker CLI** commands. This approach involved:
- Manually creating networks.
- Individually building and running each container.
- Managing dependencies between services manually.

This becomes complex and hard to manage, especially when deploying at scale.

### **Solution: Use Docker Compose**
Docker Compose simplifies multi-container applications by allowing you to:
- Define services, networks, and volumes in a single YAML file.
- Start and stop the entire application with a single command.
- Automate dependencies (e.g., backend waits for the database).
- Improve maintainability.

---

## **Cleanup Guide before we start**  

## **1. Remove All Stopped Containers**  
Before removing all images, it is recommended to **remove stopped containers** to free up space.  

### **Check for Stopped Containers**  
Run the following command to list all stopped containers:  
```sh
docker ps -a
```  
If there are containers that you no longer need, remove them using:  
```sh
docker container prune -f
```  
The `-f` flag forces removal **without confirmation**. If you want a prompt before deletion, remove `-f`.  

---

## **2. Stop and Remove All Running Containers**  
To completely clean up running containers:  

### **List Running Containers**  
```sh
docker ps
```  
### **Stop All Running Containers**  
```sh
docker stop $(docker ps -q)
```  
### **Remove All Containers (Stopped + Running)**  
```sh
docker rm $(docker ps -aq)
```  

---

## **3. Remove All Docker Images**  
After stopping and removing containers, remove all images to free up space.  

### **List All Images**  
```sh
docker images
```  
### **Remove All Images**  
```sh
docker rmi -f $(docker images -q)
```  

---

## **4. Remove All Unused Docker Data (Volumes, Networks, and Build Cache)**  
To completely clean up the system, remove unused volumes, networks, and build cache.  

### **Remove Unused Volumes**  
```sh
docker volume prune -f
```  
### **Remove Unused Networks**  
```sh
docker network prune -f
```  
### **Remove Build Cache (Dangling Images, Unused Layers)**  
```sh
docker system prune -a -f
```  
This command removes **all stopped containers, unused networks, dangling images, and build cache**.  

---

## **Final Verification**  
Run the following commands to confirm everything is removed:  

- **Check that no containers are running or stopped:**  
  ```sh
  docker ps -a
  ```  
- **Check that no images exist:**  
  ```sh
  docker images
  ```  
- **Check that no volumes exist:**  
  ```sh
  docker volume ls
  ```  

## **Step 1: Define the Project Structure**
Create the following folder structure:

```sh
mkdir -p multi-tier-app/{frontend,backend,database}
cd multi-tier-app
touch docker-compose.yml
```

Final structure:
```
multi-tier-app/
│-- frontend/
│   └── Dockerfile
│-- backend/
│   └── Dockerfile
│-- database/
│   └── Dockerfile
│-- docker-compose.yml
```

---

## **Step 2: Write the `docker-compose.yml` File**
Inside `multi-tier-app/docker-compose.yml`, add the following configuration:

```yaml
version: "3.8"

services:
  database:
    build: ./database
    container_name: db
    restart: always
    networks:
      - backend-network
    volumes:
      - mongo-data:/data/db
    ports:
      - "27017:27017"

  backend:
    build: ./backend
    container_name: api
    restart: always
    depends_on:
      - database
    networks:
      - backend-network
      - frontend-network

  frontend:
    build: ./frontend
    container_name: ui
    restart: always
    depends_on:
      - backend
    networks:
      - frontend-network
    ports:
      - "80:80"

networks:
  backend-network:
  frontend-network:

volumes:
  mongo-data:
```

---

## **Step 3: Create the Dockerfiles**
### **Database (MongoDB)**
Inside `multi-tier-app/database/Dockerfile`:
```dockerfile
# Use the official MongoDB image
FROM mongo:latest

# Expose MongoDB's default port
EXPOSE 27017
```

---

### **Backend (Node.js with Express)**
Inside `multi-tier-app/backend/Dockerfile`:
```dockerfile
# Use the official Node.js image
FROM node:18

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm init -y && npm install express mongoose cors body-parser

# Copy application code
COPY . .

# Create index.js with Express API and MongoDB connection
RUN echo "const express = require('express'); \
const mongoose = require('mongoose'); \
const app = express(); \
const PORT = 80; \
mongoose.connect('mongodb://db:27017/mydb', { useNewUrlParser: true, useUnifiedTopology: true }) \
.then(() => console.log('Connected to MongoDB')) \
.catch(err => console.log('MongoDB connection error:', err)); \
app.get('/', (req, res) => res.send('Hello from Backend')); \
app.listen(PORT, () => console.log('Backend running on port ' + PORT));" > index.js

# Expose port 80
EXPOSE 80

# Run the application
CMD ["node", "index.js"]
```

---

### **Frontend (React with Nginx)**
Inside `multi-tier-app/frontend/Dockerfile`:
```dockerfile
# Use the official lightweight Nginx image
FROM nginx:alpine

# Expose port 80 for incoming traffic
EXPOSE 80

# Start Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]
```

---

## **Step 4: Build and Start the Application**

**Install docker-compose**

```sh
sudo apt  install docker-compose
```


Navigate to `multi-tier-app/` and run:
```sh
docker-compose up --build -d
```
- `--build` forces a rebuild of images.
- `-d` runs services in the background.

---

## **Step 5: Verify Everything is Running**
### **Check Running Containers**
```sh
docker ps
```
Expected output:
```
CONTAINER ID   IMAGE           STATUS         NAMES
a1b2c3d4e5f6   frontend-app    Up 10 min     ui
b2c3d4e5f6a1   backend-app     Up 10 min     api
c3d4e5f6a1b2   mongo           Up 10 min     db
```

---

## **Step 6: Test the Application**
### **Check Frontend (React)**
Since the frontend is exposed on **port 80**, test it:
```sh
curl http://localhost
```
Expected output:
```
Welcome to nginx!
```

### **Check Backend API**
Since the backend is also exposed on **port 80**, test it:
```sh
curl http://localhost
```
Expected output:
```
Hello from Backend
```

### **Check Database Connection**
Access the backend container:
```sh
docker exec -it api /bin/sh
```
Inside the backend container, run:
```sh
mongosh --host db --port 27017
```
Expected output:
```
Current MongoDB server version: 6.x.x
Connecting to: mongodb://db:27017/
```
Exit:
```sh
exit
```

---

## **Step 7: Manage Services**
### **Restart the Backend**
```sh
docker-compose restart backend
```

### **Stop the Entire Application**
```sh
docker-compose down
```

---

## **Key Benefits of Using Docker Compose**
| Feature | Without Compose | With Compose |
|------------|----------------------|------------------|
| **Setup Complexity** | Manual container creation & linking | Defined in a single `docker-compose.yml` file |
| **Dependency Handling** | Must manually start services in order | `depends_on` ensures proper startup |
| **Networking** | Manually create networks | Networks are defined in Compose |
| **Storage** | Manually attach volumes | Managed by Compose |
| **Scalability** | Difficult to scale | Easily scale services with `docker-compose up --scale backend=3` |
