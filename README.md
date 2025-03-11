# Deploying and Sharing a Dockerized React Application

This repository provides a **step-by-step guide** to deploy, Dockerize, and share a React application using Docker and Nginx on an Ubuntu VM. Additionally, it covers **Docker networking**, allowing students to understand how containers communicate in different environments.

Each section is covered in a separate file for structured learning.

---

## **Table of Contents**

### **1. Install Docker on Ubuntu**
- Installing Docker on an Ubuntu VM
- Starting and enabling Docker

📄 [Install Docker](01_Install_Docker.md)

---

### **2. Deploy React Application Using Nginx**
- Installing Nginx
- Building and deploying a React application
- Configuring Nginx to serve the application
- Accessing the application from a public IP

📄 [Deploy React Application Using Nginx](02_React_application_using_Nginx.md)

---

### **3. Dockerizing the React Application**
- Writing a Dockerfile for the React application
- Building and running the Docker container
- Exposing the application to the host machine

📄 [Dockerizing the React Application](03_Dockerizing_React_Application.md)

---

### **4. Sharing the Docker Image on Docker Hub**
- Creating a Docker Hub account
- Tagging and pushing the image to Docker Hub
- Running the image from Docker Hub on any machine

📄 [Sharing the Docker Image](04_Sharing_Docker_Image.md)

---

### **5. Docker Networking**
**Understanding how containers communicate** is crucial for building scalable applications. This section explores different networking modes in Docker and how they impact container communication.

#### **Topics Covered:**
1. **Scenario 1: Running a Standalone Application**
   - Deploying a single container
   - Mapping ports for external access

2. **Scenario 2: Two Containers Communicating (Microservices)**
   - Using a custom **bridge network** for inter-container communication
   - Connecting a frontend container to a backend API

3. **Scenario 3: Multi-Tier Architecture (Frontend, Backend, Database)**
   - Creating isolated networks for better **security and performance**
   - Connecting a **backend service** to a **database container**
   - Allowing frontend-to-backend communication without direct database exposure

4. **Scenario 4: Using Host Mode for High-Performance Applications**
   - Running a container in **host mode** for low-latency applications
   - Understanding the impact of **NAT elimination** and **direct network access**

📄 [Docker Networking](05_Docker_Networking.md)

---

## **Key Takeaways**
- **Deploy and Dockerize applications** efficiently.
- **Use Nginx** to serve web applications.
- **Share and distribute** Docker images via Docker Hub.
- **Understand container networking** to build scalable and secure architectures.

---

This structured approach ensures a **step-by-step learning experience**, helping users **progress from basic deployments to advanced networking concepts**.

