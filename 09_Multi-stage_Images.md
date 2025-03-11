## **Tutorial: Multi-Stage Build with Docker (Optimizing the Frontend Container)**  

### **Problem Statement**  
In our previous **frontend Dockerfile**, we used **Nginx** to serve a React application. However, the Docker image still contained unnecessary dependencies, increasing the **image size** and making it inefficient.  

A **multi-stage build** allows us to:  
- Build the React application inside a **Node.js container** (with dependencies).  
- Copy only the **optimized production build** to a **lightweight Nginx container**.  
- Reduce the final image size and improve security.  

---

## **Step 1: Comparing Single-Stage and Multi-Stage Builds**  

Before using a **multi-stage build**, let's create a **single-stage** Dockerfile and check the image size.  

### **Single-Stage Dockerfile (Inefficient Approach)**  

Inside `multi-tier-app/frontend/Dockerfile`, add the following:  

```dockerfile
# Use Node.js as the base image
FROM node:18

# Set the working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy all application files
COPY . .

# Build the application
RUN npm run build

# Expose port 80
EXPOSE 80

# Run the application
CMD ["npm", "start"]
```

### **Build and Check Image Size**
```sh
docker build -t frontend-single-stage ./frontend
docker images frontend-single-stage
```
Expected output:
```
REPOSITORY              TAG       IMAGE ID       SIZE
frontend-single-stage   latest    abc123def456   1.2GB
```
The image is large because it contains:  
- The entire **Node.js runtime**.  
- Development dependencies, even though they are not needed in production.  

---

## **Step 2: Optimizing with Multi-Stage Build**  

A **multi-stage Dockerfile** consists of:  
1. **Stage 1 (Build Stage)** → Uses **Node.js** to install dependencies and create a production-ready build.  
2. **Stage 2 (Runtime Stage)** → Uses **Nginx** to serve the optimized React build, without unnecessary files.  

---

## **Step 3: Multi-Stage Dockerfile**  

Modify `multi-tier-app/frontend/Dockerfile` to use **multi-stage builds**:  

```dockerfile
# Stage 1: Build the React Application
FROM node:18 AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy the entire project and build the app
COPY . .
RUN npm run build

# Stage 2: Serve the Application with Nginx
FROM nginx:alpine

# Set working directory in Nginx
WORKDIR /usr/share/nginx/html

# Remove default Nginx static files
RUN rm -rf ./*

# Copy the built application from the first stage
COPY --from=builder /app/build .

# Expose port 80 for incoming traffic
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

---

## **Step 4: Explaining the Multi-Stage Build (Line by Line)**  

### **Stage 1: Build the React Application**  

```dockerfile
FROM node:18 AS builder
```
- Uses **Node.js 18** as the base image.
- The `AS builder` keyword defines this as the **build stage**.

```dockerfile
WORKDIR /app
```
- Sets the working directory inside the container.

```dockerfile
COPY package.json package-lock.json ./
RUN npm install
```
- Copies the **package.json** files first to leverage **Docker's caching mechanism**.  
- Installs dependencies **before copying the rest of the code**.

```dockerfile
COPY . .
RUN npm run build
```
- Copies all the application files.
- Runs `npm run build` to generate the **production-ready React application**.

---

### **Stage 2: Serve the Application with Nginx**  

```dockerfile
FROM nginx:alpine
```
- Uses **Nginx Alpine** as the final lightweight image (small and optimized for serving static files).

```dockerfile
WORKDIR /usr/share/nginx/html
```
- Sets the working directory to Nginx’s default static file location.

```dockerfile
RUN rm -rf ./*
```
- Removes the default Nginx static files.

```dockerfile
COPY --from=builder /app/build .
```
- Copies the production-ready **build files** from the `builder` stage.  
- This eliminates unnecessary development files and dependencies.

```dockerfile
EXPOSE 80
```
- Exposes port **80** to serve the application.

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```
- Runs **Nginx** in the foreground to serve the static files.

---

## **Step 5: Build and Run the Optimized Image**  
Navigate to `multi-tier-app/` and build the **optimized frontend image**:  
```sh
docker build -t frontend-multi-stage ./frontend
```

Run the **container**:
```sh
docker run -d --name ui -p 80:80 frontend-multi-stage
```

Verify that the **frontend application is accessible**:
```sh
curl http://localhost
```

---

## **Step 6: Compare Image Sizes**  
Check the size of the optimized image:
```sh
docker images frontend-multi-stage
```
Expected output:
```
REPOSITORY              TAG       IMAGE ID       SIZE
frontend-multi-stage    latest    xyz789ghi123   25MB
```

### **Size Comparison Table**
| Build Type | Image Size |
|------------|------------|
| **Single-Stage (Node.js)** | 1.2GB |
| **Multi-Stage (Optimized)** | 25MB |

By using a multi-stage build, the final image is **significantly smaller** because:  
- The **Node.js runtime** is no longer included.  
- Only the **production build** is copied to the final image.  

---

## **Key Benefits of Multi-Stage Builds**
| Feature | Without Multi-Stage | With Multi-Stage |
|---------|---------------------|------------------|
| **Image Size** | Large (includes Node.js & dependencies) | Small (only contains optimized React build) |
| **Security** | Exposes unnecessary tools | Minimal attack surface |
| **Performance** | Slower due to large image | Faster startup & deployment |
| **Best For** | Development environments | Production deployments |


