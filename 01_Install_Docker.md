Here’s a step-by-step **Ubuntu-only** guide for installing and setting up Docker.  

---

# **Installing Docker on Ubuntu (Step-by-Step Guide)**  

Docker is a containerization platform that allows you to package applications and their dependencies into isolated environments called **containers**. This guide will show you how to install and configure **Docker on Ubuntu**.

---

## **1. Update Your System**  
Before installing Docker, update your package list to ensure you are installing the latest versions of dependencies.

Open the terminal and run:
```sh
sudo apt update && sudo apt upgrade -y
```

---

## **2. Install Required Dependencies**  
Install the required packages that allow Ubuntu to fetch software from external repositories:
```sh
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

---

## **3. Add Docker’s Official GPG Key**  
To ensure package security, add Docker’s official **GPG key**:
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

---

## **4. Add Docker Repository**  
Now, add the **Docker repository** to Ubuntu’s package manager:
```sh
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

---

## **5. Install Docker**  
Update the package list again and install **Docker Engine**:
```sh
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

---

## **6. Verify Docker Installation**  
To check if Docker is installed correctly, run:
```sh
docker --version
```
You should see output similar to:
```
Docker version 24.0.5, build ced0996
```

---

## **7. Start and Enable Docker**  
To start Docker and enable it to launch on boot, run:
```sh
sudo systemctl start docker
sudo systemctl enable docker
```

Verify if Docker is running:
```sh
sudo systemctl status docker
```
If running, you will see **"active (running)"** in green.

**Test Docker:** 

```sh
docker run hello-world
```

You will get error:

```sh
docker: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
```

---

## **8. Run Docker Without `sudo` (Optional but Recommended)**  
By default, Docker commands require **sudo**. To run Docker without sudo, add your user to the **docker group**:

```sh
sudo usermod -aG docker $USER
```

Now, **log out and log back in**, or run:
```sh
newgrp docker
```

Try running Docker without sudo:
```sh
docker run hello-world
```
If successful, you will see:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## **9. Test Docker Installation**  
Run the following command to check if Docker is working:
```sh
docker run hello-world
```
This downloads a small test container and runs it.
