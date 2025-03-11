# **Multi-Stage Build with a Go Application**  

## **Problem Statement**  
A **single-stage build** for a Go application:
- Produces **large images** because it includes compilers and dependencies.
- Includes **unnecessary files** in production.
- Slows down deployment.

A **multi-stage build** allows us to:
- **Build the application in a full-featured development container**.
- **Copy only the final binary to a minimal runtime container**.
- **Reduce the final image size**, making it more efficient for deployment.

---

## **Step 1: Create a Simple Go Application**  
Navigate to your project folder and create a new directory:  
```sh
mkdir go-multi-stage && cd go-multi-stage
```

Create a **main.go** file:
```sh
touch main.go
```

Add the following Go code inside `main.go`:
```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from Multi-Stage Build!")
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Server started on port 80")
    http.ListenAndServe(":80", nil)
}
```

---

# **Step 2: Single-Stage Build (Without Optimization)**
This is the **inefficient approach**, where we use a single container for **both compilation and runtime**.

### **Single-Stage Dockerfile**
Inside `go-multi-stage/`, create a **Dockerfile**:
```sh
touch Dockerfile
```

Add the following content:
```dockerfile
# Use the full Go image with build tools
FROM golang:1.19

# Set the working directory inside the container
WORKDIR /app

# Copy the Go source code
COPY main.go .

# Compile the Go application
RUN go mod init app && go mod tidy && go build -o app

# Expose port 80 for incoming traffic
EXPOSE 80

# Run the application
CMD ["./app"]
```

---

### **Step 3: Build and Run the Single-Stage Image**
Build the **single-stage** image:
```sh
docker build -t go-single-stage .
```

Check the **image size**:
```sh
docker images go-single-stage
```

**Expected Output:**
```
REPOSITORY        TAG       IMAGE ID       SIZE
go-single-stage   latest    abc123def456   900MB
```
The image is **very large** because:
- It **includes the full Go compiler and dependencies**.
- It is **not optimized for production use**.

Run the container:
```sh
docker run -d --name go-app-single -p 80:80 go-single-stage
```

Test if it works:
```sh
http://<PublicIP>
```

**Expected Output:**
```
Hello from Multi-Stage Build!
```

---

# **Step 4: Multi-Stage Build (Optimized Approach)**
Now, let's optimize the image by using a **multi-stage build**.

### **Multi-Stage Dockerfile**
Modify `Dockerfile` as follows:

```dockerfile
# Stage 1: Build the Go Application
FROM golang:1.19 AS builder

# Set working directory inside the container
WORKDIR /app

# Copy the Go source code
COPY main.go .

# Build the Go application with static linking
RUN go mod init app && go mod tidy && \
    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o app

# Stage 2: Create a Minimal Runtime Image
FROM alpine:latest

# Set working directory
WORKDIR /root/

# Copy only the compiled binary from the builder stage
COPY --from=builder /app/app .

# Expose port 80 for incoming traffic
EXPOSE 80

# Run the application
CMD ["./app"]
```

---

### **Step 5: Build and Run the Optimized Multi-Stage Image**
Build the **optimized multi-stage** image:
```sh
docker build -t go-multi-stage .
```

Check the **image size**:
```sh
docker images go-multi-stage
```

**Expected Output:**
```
REPOSITORY       TAG       IMAGE ID       SIZE
go-multi-stage   latest    xyz789ghi123   10MB
```

### **Why is the Image Much Smaller?**
- **Only the compiled Go binary is included**.
- **No need for compilers or extra tools** in production.
- Uses **Alpine Linux**, which is much smaller than the full Go image.

---

### **Step 6: Compare Image Sizes**
Now, let's **compare single-stage and multi-stage images**:

| Build Type | Image Size |
|------------|------------|
| **Single-Stage (Go with Build Tools)** | 900MB |
| **Multi-Stage (Optimized)** | 10MB |

---

### **Step 7: Run and Test the Optimized Multi-Stage Image**
Run the optimized container:
```sh
docker run -d --name go-app -p 80:80 go-multi-stage
```

Test if it works:
```sh
http://<PublicIP>
```

**Expected Output:**
```
Hello from Multi-Stage Build!
```

---

## **Key Benefits of Multi-Stage Builds**
| Feature | Without Multi-Stage | With Multi-Stage |
|---------|---------------------|------------------|
| **Image Size** | Large (includes Go compiler & dependencies) | Small (only contains final binary) |
| **Security** | Exposes unnecessary tools | Minimal attack surface |
| **Performance** | Slower due to large image | Faster startup & deployment |
| **Best For** | Development environments | Production deployments |

---

