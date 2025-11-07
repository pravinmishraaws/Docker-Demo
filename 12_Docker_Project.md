# **Azure VM Setup and Application Deployment Guide**

## **1. Launch an Azure Virtual Machine (VM)**
Follow these steps to create and configure an Azure Virtual Machine (VM) to deploy your application.

### **VM Configuration:**
- **Authentication Method:** Use **password-based authentication** (**NOT** SSH key).
- **Allow Inbound Ports:** Ensure the following ports are open:
  - **Port 22** (for SSH access)
  - **Port 80** (for HTTP traffic)
- **Public IP:** Assign a **public IP** to the VM.
- **Custom Data (Cloud-Init Script):** In the **Advanced** settings section, add the following script to install **Docker** automatically upon VM creation.

### **Custom Data (User Data) Script for Docker Installation**
Copy and paste this script into the **Custom Data** field during VM setup:

<img width="912" alt="Custom Data" src="https://github.com/user-attachments/assets/7201b950-519e-4603-a611-ef4aa381760d" />


```bash
#!/bin/bash
apt update && apt upgrade -y
apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list
apt update
apt install -y docker-ce docker-ce-cli containerd.io
systemctl start docker
systemctl enable docker

```
- Click **Review + Create** and launch the VM.

---

## **2. Configure Network Security Rules**
After the VM is launched, update its **Network Security Group (NSG)** to allow traffic for your application.

<img width="1287" alt="Screenshot 2025-03-14 at 21 36 45" src="https://github.com/user-attachments/assets/db820136-6df1-4365-aba7-1c8ccff556a8" />


### **Steps to Open Required Ports**
1. **Add Inbound Rules**:
   - Allow **Port 3000** (for application access).
  
     <img width="401" alt="Screenshot 2025-03-14 at 21 37 25" src="https://github.com/user-attachments/assets/4f60dd4a-8749-4566-a76c-1faaaa8d2d5b" />


2. **Add Outbound Rules**:

   - Allow **Port 3000** to enable outgoing traffic.
  
     <img width="1031" alt="Screenshot 2025-03-14 at 21 37 58" src="https://github.com/user-attachments/assets/5747a5ce-3910-4c4b-b366-0fcea66a40ae" />


Refer to the Azure **Network Settings** UI to apply these changes.

---

## **3. Transfer Application Code to the VM**
Once the VM is running and network rules are configured, upload your application code to the VM.

[Download the Appliation code](https://github.com/pravinmishraaws/Azure-Static-Website)

### **Steps to Transfer Code:**
1. Open a terminal on your local machine.
2. Navigate to the directory where your application code is located.
3. Use `scp` to securely transfer the code to the VM:
   ```sh
   scp -r * username@<PublicIP>:/home/username
   ```
   Replace `<PublicIP>` with your VM’s **public IP address**.

4. **SSH into the VM:**
   ```sh
   ssh username@<PublicIP>
   ```

---

## **4. Grant Docker Permissions to the Current User**
After logging into the VM, allow your user to run Docker commands without `sudo`.

```sh
sudo usermod -aG docker $USER
newgrp docker
docker ps  # Verify Docker is running
```

---

## **5. Update Frontend Configuration**
Modify your UI configuration file (`ui/config.json`) to point to the VM’s **public IP**.

```json
{
    "API_URL": "http://<PublicIP>:3000/"
}
```
Replace `<PublicIP>` with your VM’s actual **public IP address**.

---

## **6. Install Docker Compose & Start Containers**
Install Docker Compose and start the application containers.

```sh
sudo apt install -y docker-compose
```

Then, navigate to the application directory and start the containers:

```sh
docker-compose up --build
```
Output

```sh
basic-3tier-ui          | 2025/03/14 20:08:30 [notice] 1#1: start worker process 22
basic-3tier-api         | 
basic-3tier-api         | > backend3tiernode@1.0.0 start
basic-3tier-api         | > node app.js
basic-3tier-api         | 
basic-3tier-api         | (node:19) [MODULE_TYPELESS_PACKAGE_JSON] Warning: Module type of file:///usr/src/app/app.js is not specified and it doesn't parse as CommonJS.
basic-3tier-api         | Reparsing as ES module because module syntax was detected. This incurs a performance overhead.
basic-3tier-api         | To eliminate this warning, add "type": "module" to /usr/src/app/package.json.
basic-3tier-api         | (Use `node --trace-warnings ...` to show where the warning was created)
basic-3tier-api         | Connected to PostgreSQL Database
basic-3tier-api         | Database initialized
basic-3tier-api         | Server running on port 3000
```

---

## **7. Verify Deployment**
1. **Check running containers:**
   ```sh
   docker ps
   ```
2. **Open the application in a browser:**
   ```
   http://<PublicIP>:80/
   ```

---

## **Troubleshooting**
- If Docker containers are not running, check logs:
  ```sh
  docker logs <container_name>
  ```
