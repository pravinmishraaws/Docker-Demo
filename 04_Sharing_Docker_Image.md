# **Sharing the Docker Container on Docker Hub**  

In this, you will **publish your Dockerized React application** to **Docker Hub** so that others can easily download and run it on their own machines.  

---

## **Step 1: Create a Docker Hub Account**  

To share your Docker image, you first need a **Docker Hub account**.  

### **How to Create an Account:**  
1. Go to **[Docker Hub](https://hub.docker.com/)**  
2. Click **"Sign Up"**  
3. Fill in your details and create an account.  
4. Once signed in, create a **new repository**:
   - Click **"Create Repository"**  
   - Set **Repository Name** (e.g., `my-react-app`)  
   - Choose **Public** (if you want to share it) or **Private**  

---

## **Step 2: Log in to Docker Hub from Your Ubuntu VM**  

Before pushing an image, you need to log in to **Docker Hub** from your terminal:  

**Note:** You must be in your Ubuntu VM.  

```sh
docker login
```

### **Explanation:**
- This command prompts you to enter your **Docker Hub username and password**.
- Once logged in successfully, you will see a message:  
  ```
  Login Succeeded
  ```

---

## **Step 3: Tag the Docker Image**  

Before pushing an image to Docker Hub, you need to **tag** it with your **Docker Hub username and repository name**.  

### **Tagging the Image:**
```sh
docker tag my-react-app <your-dockerhub-username>/my-react-app:latest
```

### **Explanation:**
- `docker tag my-react-app` → Specifies the **existing local image** you built.  
- `<your-dockerhub-username>/my-react-app:latest` → Creates a new **tag** that includes your **Docker Hub username**.  
- `latest` → Specifies the **version tag** (default tag).  

Example (if your username is `pravinmishraaws`):  
```sh
docker tag my-react-app pravinmishraaws/my-react-app:latest
```

---

## **Step 4: Push the Image to Docker Hub**  

Now, upload the image to your Docker Hub repository using:  

```sh
docker push <your-dockerhub-username>/my-react-app:latest
```

Example (if your username is `pravinmishraaws`):  
```sh
docker push pravinmishraaws/my-react-app:latest
```

### **Explanation:**
- `docker push` → Uploads the **tagged image** to Docker Hub.
- `<your-dockerhub-username>/my-react-app:latest` → Specifies the **repository and tag**.

Once the **push is complete**, you can see the image in your **Docker Hub repository** at:  

```
https://hub.docker.com/r/<your-dockerhub-username>/my-react-app
```

Example (if your username is `pravinmishraaws`):  
```
https://hub.docker.com/r/pravinmishraaws/my-react-app
```

---

# **Your Docker Image is Now Publicly Available**  

Now, anyone can **pull and run** your React application using:  

```sh
docker pull <your-dockerhub-username>/my-react-app:latest
docker run -d -p 80:80 --name react-container <your-dockerhub-username>/my-react-app:latest
```

Example (for `pravinmishraaws`):  
```sh
docker pull pravinmishraaws/my-react-app:latest
docker run -d -p 80:80 --name react-container pravinmishraaws/my-react-app:latest
```

This will **download and run your React app on any machine**.  
