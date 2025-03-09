## **Dockerizing and Running the React Application on Ubuntu VM**  

This guide will help you **Dockerize** the **React application** from the **GitHub repository** and deploy it using **Docker**, making it accessible from a **public IP**.

---

## **1. Remove Any Existing Files from Nginx Web Directory**  
Since we are running this as a **Docker container**, we don't need old files inside `/var/www/html/`. **Remove them**:

```sh
sudo rm -rf /var/www/html/*
```

---

## **2. Create a `Dockerfile`**  
Inside the **project root directory**, create a file named **Dockerfile**:  

```sh
cd my-react-app/
vi Dockerfile
```

Paste the following content into the file:  

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

Save and exit (`CTRL + X`, then `Y`, then `ENTER`).

---

## **3. Build the Docker Image**  
Now, build the **Docker image** for the React app:  

```sh
docker build -t my-react-app .
```

This will:  
- **Install dependencies**
- **Build the React app**
- **Copy the built files to an Nginx container**  

---

## **4. Run the Docker Container and Expose it to the Host**  
Now, **run the container** and expose it on **port 80** so it’s accessible from the **public IP**:  

```sh
docker run -d -p 80:80 --name react-container my-react-app
```

This maps **port 80 inside the container** to **port 80 on the host machine**.

---

## **5. Verify the Running Docker Container**  
To check if the container is running:  

```sh
docker ps
```

If needed, check the logs:  

```sh
docker logs react-container
```

---

## **6. Access the React Application from Public IP**  
Find your **public IP**:  

```sh
curl ifconfig.me
```

Now, visit your React app in a browser:  

```
http://<your-public-ip>
```

For example, if your **public IP is 203.0.113.25**, then:  

```
http://203.0.113.25
```

---

## **7. Stopping and Restarting the Container**  
To **stop** the container:  

```sh
docker stop react-container
```

To **start** the container again:  

```sh
docker start react-container
```

To **remove** the container:  

```sh
docker rm -f react-container
```

To **remove the image**:  

```sh
docker rmi my-react-app
```

---

## **Your React App is Now Dockerized and Running on Ubuntu VM!**  
Now your **React application** runs inside a **Docker container** using **Nginx**, accessible from a **public IP**.
