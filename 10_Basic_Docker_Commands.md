Here is the **complete list of Docker commands** used in this course, **organized by topic**, with **missing commands added** and **no emojis**.

---

# **Basic Docker Commands**
### **Installing & Verifying Docker**
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install docker-ce docker-ce-cli containerd.io -y
docker --version
sudo systemctl start docker
sudo systemctl enable docker
```

### **Check Docker Status**
```sh
sudo systemctl status docker
```

### **Run Docker Without Sudo**
```sh
sudo usermod -aG docker $USER
newgrp docker
```

### **Verify Docker Installation**
```sh
docker run hello-world
```

---

# **Working with Containers**
### **Run a Container**
```sh
docker run hello-world  # Run a test container
docker run -d --name my-container nginx  # Run an Nginx container in detached mode
docker run -d --name backend node:18  # Run a backend container
docker run -d --name frontend nginx  # Run a frontend container
docker run -d --name db mongo  # Run a MongoDB container
docker run -it ubuntu bash  # Run a container interactively with a terminal
```

### **Run a Container with Port Mapping**
```sh
docker run -d --name web -p 80:80 nginx
```

### **Run a Container with a Volume**
```sh
docker run -d --name app -v myvolume:/app/data nginx
```

### **List Running and Stopped Containers**
```sh
docker ps      # Show running containers
docker ps -a   # Show all containers (running + stopped)
```

### **Inspect a Container**
```sh
docker inspect my-container
```

### **View Container Logs**
```sh
docker logs my-container
docker logs -f my-container  # Follow logs in real-time
```

### **Stop and Remove Containers**
```sh
docker stop my-container
docker rm my-container
```
**Stop & Remove All Containers**
```sh
docker ps -q | xargs -r docker stop
docker ps -aq | xargs -r docker rm
```

---

# **Working with Docker Images**
### **List Images**
```sh
docker images
```

### **Pull & Remove Images**
```sh
docker pull nginx
docker rmi nginx
```
**Remove All Images**
```sh
docker images -q | xargs -r docker rmi -f
```

### **Inspect an Image**
```sh
docker image inspect nginx
```

### **Check Image History**
```sh
docker history nginx
```

---

# **Working with Docker Networks**
### **List Networks**
```sh
docker network ls
```

### **Create a Custom Network**
```sh
docker network create mynetwork
```

### **Run Containers on a Custom Network**
```sh
docker run -d --name backend --network mynetwork node:18
docker run -d --name frontend --network mynetwork nginx
```

### **Inspect a Network**
```sh
docker network inspect mynetwork
```

### **Test Network Communication**
```sh
docker exec -it frontend curl backend
```

### **Connect a Container to an Additional Network**
```sh
docker network connect frontend-network backend
```

### **Remove Networks**
```sh
docker network rm mynetwork
```
**Remove All Custom Networks**
```sh
docker network ls --format "{{.Name}}" | grep -Ev "bridge|host|none" | xargs -r docker network rm
```

---

# **Working with Volumes**
### **List, Create & Remove Volumes**
```sh
docker volume ls
docker volume create myvolume
docker volume rm myvolume
```

### **Run a Container with a Volume**
```sh
docker run -d -v myvolume:/app/data nginx
```

### **Inspect a Volume**
```sh
docker volume inspect myvolume
```

### **Remove All Volumes**
```sh
docker volume ls -q | xargs -r docker volume rm
```

---

# **Dockerizing Applications**
### **Building a Docker Image**
```sh
docker build -t my-app .
```

### **Run a Container from a Custom Image**
```sh
docker run -d --name my-container -p 80:80 my-app
```

### **Tag an Image**
```sh
docker tag my-app myrepo/my-app:latest
```

### **Push an Image to Docker Hub**
```sh
docker login
docker push myrepo/my-app:latest
```

### **Pull an Image from Docker Hub**
```sh
docker pull myrepo/my-app:latest
```

---

# **Docker Compose**
### **Starting & Stopping Services**
```sh
docker-compose up -d  # Start services in detached mode
docker-compose down   # Stop and remove services
```

### **Rebuild and Restart Services**
```sh
docker-compose up --build -d
```

### **Check Running Services**
```sh
docker-compose ps
```

### **View Logs**
```sh
docker-compose logs -f
```

### **Run a Command Inside a Service**
```sh
docker-compose exec backend bash
```

---

# **Multi-Stage Build (Optimization)**
### **Single-Stage Build**
```sh
docker build -t go-single-stage .
docker run -d --name go-app -p 80:80 go-single-stage
```

### **Multi-Stage Build**
```sh
docker build -t go-multi-stage .
docker run -d --name go-app -p 80:80 go-multi-stage
```

---

# **Full Docker Cleanup**
### **Remove Everything (Containers, Images, Volumes, Networks)**
```sh
docker ps -q | xargs -r docker stop
docker ps -aq | xargs -r docker rm
docker images -q | xargs -r docker rmi -f
docker volume ls -q | xargs -r docker volume rm
docker network ls --format "{{.Name}}" | grep -Ev "bridge|host|none" | xargs -r docker network rm
docker system prune -a --volumes -f  # Cleanup all unused resources
```

### **Verify Cleanup**
```sh
docker ps -a        # Should return nothing (no containers)
docker images       # Should return nothing (no images)
docker volume ls    # Should return nothing (no volumes)
docker network ls   # Should only show default networks (bridge, host, none)
```
