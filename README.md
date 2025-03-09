# **Deploying and Sharing a Dockerized React Application**  

This repository provides a **step-by-step guide** to **deploy, Dockerize, and share a React application** using **Docker and Nginx** on an **Ubuntu VM**.  

Each section is covered in a separate file for structured learning.  

---

## **Table of Contents**  

1. **[Install Docker on Ubuntu](01_Install_Docker.md)**  
   - Installing Docker on an Ubuntu VM  
   - Starting and enabling Docker  

2. **[Deploy React Application Using Nginx](02_React_application_using_Nginx.md)**  
   - Installing Nginx  
   - Building and deploying a React application  
   - Configuring Nginx to serve the application  
   - Accessing the application from a public IP  

3. **[Dockerizing the React Application](03_Dockerizing_React_Application.md)**  
   - Writing a `Dockerfile` for the React application  
   - Building and running the Docker container  
   - Exposing the application to the host machine  

4. **[Sharing the Docker Image on Docker Hub](04_Sharing_Docker_Image.md)**  
   - Creating a Docker Hub account  
   - Tagging and pushing the image to Docker Hub  
   - Running the image from Docker Hub on any machine  
