## **Optimizing the Backend with Multi-Stage Build in Docker**

### **Problem Statement**
Previously, the backend service was built using a **single-stage Dockerfile**. This approach:
- Increased the image size because it contained **unnecessary build dependencies**.
- Included development tools, making the final image less secure.
- Made deployment inefficient by storing everything inside a single large container.

A **multi-stage build** allows us to:
- Build the backend application inside a **Node.js container** with all dependencies.
- Copy only the **necessary runtime files** to a **lightweight Node.js container**.
- Reduce the final image size and improve security.

---

## **Step 1: Single-Stage Dockerfile (Before Optimization)**  

Inside `multi-tier-app/backend/Dockerfile`, the original **single-stage** build looked like this:

```dockerfile
# Use the official Node.js image
FROM node:18

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
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

### **Build and Check Image Size**
Navigate to the `multi-tier-app/` directory and build the **single-stage backend image**:

```sh
docker build -t backend-single-stage ./backend
```

Now, check the image size:
```sh
docker images backend-single-stage
```

Expected output:
```
REPOSITORY              TAG       IMAGE ID       SIZE
backend-single-stage   latest    abc123def456   1GB
```

### **Problems with This Approach**
1. **Large Image Size** → The final image contains **Node.js and all dependencies**, making it **1GB or more**.
2. **Security Risk** → Development dependencies (e.g., `npm init -y`) are included in production.
3. **No Build Separation** → All stages (building and running) are handled inside the same container.

---

## **Step 2: Optimized Multi-Stage Dockerfile**

Modify `multi-tier-app/backend/Dockerfile` to use **multi-stage builds**:

```dockerfile
# Stage 1: Build the Backend Application
FROM node:18 AS builder

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and install dependencies (only production dependencies)
RUN npm install --only=production

# Copy the entire application code
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

# Stage 2: Create a Lightweight Runtime Container
FROM node:18-slim

# Set the working directory
WORKDIR /app

# Copy only necessary files from the builder stage
COPY --from=builder /app .

# Expose port 80 for incoming requests
EXPOSE 80

# Run the application
CMD ["node", "index.js"]
```

---

## **Step 3: Explaining the Multi-Stage Build (Line by Line)**  

### **Stage 1: Build the Backend Application**  

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
COPY package*.json ./
RUN npm install --only=production
```
- Copies only the `package.json` files to leverage **Docker caching**.
- Installs only **production dependencies** (`--only=production`) to avoid unnecessary packages.

```dockerfile
COPY . .
```
- Copies all application files.

```dockerfile
RUN echo "const express = require('express'); \
const mongoose = require('mongoose'); \
const app = express(); \
const PORT = 80; \
mongoose.connect('mongodb://db:27017/mydb', { useNewUrlParser: true, useUnifiedTopology: true }) \
.then(() => console.log('Connected to MongoDB')) \
.catch(err => console.log('MongoDB connection error:', err)); \
app.get('/', (req, res) => res.send('Hello from Backend')); \
app.listen(PORT, () => console.log('Backend running on port ' + PORT));" > index.js
```
- Generates an Express-based API and connects it to MongoDB.

---

### **Stage 2: Create a Lightweight Runtime Container**  

```dockerfile
FROM node:18-slim
```
- Uses **Node.js 18-slim**, a smaller image that reduces size by removing unnecessary tools.

```dockerfile
WORKDIR /app
```
- Sets the working directory inside the runtime container.

```dockerfile
COPY --from=builder /app .
```
- Copies only the necessary files from the `builder` stage.

```dockerfile
EXPOSE 80
```
- Exposes port **80** to accept requests.

```dockerfile
CMD ["node", "index.js"]
```
- Runs the **Node.js backend service**.

---

## **Step 4: Build and Run the Optimized Image**  
Navigate to `multi-tier-app/` and build the **optimized backend image**:  
```sh
docker build -t backend-multi-stage ./backend
```

Run the **container**:
```sh
docker run -d --name api -p 80:80 backend-multi-stage
```

Verify that the **backend application is running**:
```sh
curl http://localhost
```
Expected output:
```
Hello from Backend
```

---

## **Step 5: Compare Image Sizes**  
Check the size of the optimized image:
```sh
docker images backend-multi-stage
```
Expected output:
```
REPOSITORY              TAG       IMAGE ID       SIZE
backend-multi-stage    latest    xyz789ghi123   60MB
```

### **Size Comparison Table**
| Build Type | Image Size |
|------------|------------|
| **Single-Stage (Node.js)** | 1GB |
| **Multi-Stage (Optimized)** | 60MB |

By using a multi-stage build, the final image is significantly smaller because:  
- The **full Node.js runtime** is no longer included.  
- Only **necessary runtime files** are copied.  
- Development dependencies are **excluded**.

---

## **Key Benefits of Multi-Stage Builds**
| Feature | Without Multi-Stage | With Multi-Stage |
|---------|---------------------|------------------|
| **Image Size** | Large (includes Node.js & dependencies) | Small (only contains optimized backend code) |
| **Security** | Exposes unnecessary tools | Minimal attack surface |
| **Performance** | Slower due to large image | Faster startup & deployment |
| **Best For** | Development environments | Production deployments |

