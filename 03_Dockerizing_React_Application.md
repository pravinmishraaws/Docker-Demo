Here’s the **final version** of your **Dockerizing and Running the React Application on Ubuntu VM** guide, with **detailed explanations** for each command so your students can understand **why and how** each step works.

---

# **Dockerizing and Running the React Application on Ubuntu VM**
This guide will help you **Dockerize** the **React application** from the **GitHub repository** and deploy it using **Docker**, making it accessible from a **public IP**.

---

## **1. Remove Any Existing Files from Nginx Web Directory**
Since we are running this as a **Docker container**, we **don’t need old files inside** `/var/www/html/`.  
To remove them, run:

```sh
sudo rm -rf /var/www/html/*
```
Stop Ngix:

```sh
sudo systemctl stop nginx
```

Make sure port 80 is not occupied:

```sh
sudo lsof -i :80
```



### **Explanation:**
- `sudo` → Runs the command as **root** (superuser) because modifying system files requires elevated permissions.  
- `rm` → Removes files or directories.  
- `-rf` → **Recursive (-r)** deletes everything inside a directory, and **Force (-f)** ensures files are deleted without confirmation.  
- `/var/www/html/*` → Specifies that **all files inside `/var/www/html/`** should be deleted.

---

## **2. Create a `Dockerfile`**
Inside the **project root directory**, create a file named **Dockerfile**:

```sh
cd my-react-app/
vi Dockerfile
```

Paste the following content into the **Dockerfile**:

```Dockerfile
# Use the official Node.js image as the base image
FROM node:18-alpine AS build

# Set the working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the entire project and build the app
COPY . .
RUN npm run build

# Use Nginx to serve the React app
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 80 to allow external access
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

Save and exit (`ESC`, then `:wq`, then `ENTER` in `vi`).

### **Explanation of the Dockerfile:**
1. **`FROM node:18-alpine AS build`**  
   - This **creates a temporary container** using **Node.js 18 (Alpine Linux)**, which is a **lightweight version** of Linux optimized for containers.  
2. **`WORKDIR /app`**  
   - Sets the working directory inside the container to `/app`.  
3. **`COPY package.json package-lock.json ./`**  
   - Copies **only `package.json` and `package-lock.json`** to install dependencies faster (avoids copying unnecessary files).  
4. **`RUN npm install`**  
   - Installs all required dependencies.  
5. **`COPY . .`**  
   - Copies the entire React app code into the `/app` directory.  
6. **`RUN npm run build`**  
   - Runs the React build process, which generates a production-ready `build/` folder.  
7. **`FROM nginx:alpine`**  
   - Starts a new container using **Nginx** as the web server.  
8. **`COPY --from=build /app/build /usr/share/nginx/html`**  
   - Copies the built React app (`build/` folder) from the Node.js container to the **Nginx web root (`/usr/share/nginx/html`)**.  
9. **`EXPOSE 80`**  
   - Exposes **port 80**, allowing external access to the container.  
10. **`CMD ["nginx", "-g", "daemon off;"]`**  
    - Runs **Nginx in the foreground** so the container doesn’t exit immediately.

---

## **3. Build the Docker Image**
Now, **build the Docker image** for the React app:

```sh
docker build -t my-react-app .
```

### **Explanation:**
- `docker build` → Creates a new Docker image from the `Dockerfile`.  
- `-t my-react-app` → Assigns the image a **tag (`my-react-app`)** for easy identification.  
- `.` → Specifies that the **Dockerfile is in the current directory**.

---

## **4. Run the Docker Container and Expose It to the Host**
Now, **run the container** and expose it on **port 80** so it’s accessible from the **public IP**: 

```sh
docker run -d -p 80:80 --name react-container my-react-app
```

### **Explanation:**
- `docker run` → Starts a new **Docker container** from an image.  
- `-d` → Runs the container **in detached mode** (in the background).  
- `-p 80:80` → Maps **port 80 of the host machine** to **port 80 inside the container**, making it accessible from outside.  
- `--name react-container` → Assigns a **name** (`react-container`) to the container for easy management.  
- `my-react-app` → The **name of the Docker image** to use.

---

## **5. Verify the Running Docker Container**
To **check if the container is running**, use:

```sh
docker ps
```

### **Explanation:**
- `docker ps` → Lists all **running containers** along with their **IDs, ports, and status**.

If you need to **check logs**:

```sh
docker logs react-container
```

### **Explanation:**
- `docker logs react-container` → Fetches logs from the **react-container** to troubleshoot issues.

---

## **6. Access the React Application from Public IP**
Find your **public IP**:

```sh
curl ifconfig.me
```

Now, visit your **React app in a browser** using:

```
http://<your-public-ip>
```

For example, if your **public IP is `203.0.113.25`**, then:

```
http://203.0.113.25
```

### **Explanation:**
- `curl ifconfig.me` → Fetches the **public IP** of the Ubuntu VM.

---

## **7. Stopping and Restarting the Container**
To **stop the running container**:

```sh
docker stop react-container
```

### **Explanation:**
- `docker stop react-container` → Gracefully **stops the container** but does not remove it.

To **start the container again**:

```sh
docker start react-container
```

### **Explanation:**
- `docker start react-container` → **Restarts a stopped container** without needing to rebuild it.

To **completely remove the container**:

```sh
docker rm -f react-container
```

### **Explanation:**
- `docker rm -f react-container` → **Removes** the container **forcefully**.

To **delete the image** as well:

```sh
docker rmi my-react-app
```

### **Explanation:**
- `docker rmi my-react-app` → Removes the **Docker image** (`my-react-app`) from the system.

---

# **Your React App is Now Dockerized and Running on Ubuntu VM! **
**Your React application is now running inside a Docker container using Nginx and is accessible from a public IP.**  

This **containerized** approach makes deployment **consistent, scalable, and easier to manage**. 
