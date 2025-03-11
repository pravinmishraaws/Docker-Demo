To **completely clean up Docker** by removing **all containers, images, volumes, and networks**, run the following commands:

### **Step 1: Stop and Remove All Running and Stopped Containers**
```sh
docker ps -q | xargs -r docker stop
docker ps -aq | xargs -r docker rm
```
- `docker ps -q` → Lists all **running** container IDs.
- `docker ps -aq` → Lists **all** container IDs, including stopped ones.
- `xargs -r docker stop` → Stops all running containers.
- `xargs -r docker rm` → Removes all containers.

---

### **Step 2: Remove All Docker Networks (Except Default Networks)**
```sh
docker network ls --format "{{.Name}}" | grep -Ev "bridge|host|none" | xargs -r docker network rm

```
- Lists all networks except default ones (`bridge`, `host`, `none`) and removes them.

---

### **Step 3: Remove All Docker Volumes**
```sh
docker volume ls -q | xargs -r docker volume rm
```
- `docker volume ls -q` → Lists all volume names.
- `xargs -r docker volume rm` → Removes all volumes.

---

### **Step 4: Remove All Docker Images**
```sh
docker images -q | xargs -r docker rmi -f
```
- `docker images -q` → Lists all image IDs.
- `xargs -r docker rmi -f` → Forces removal of all images.

---

### **Step 5: Remove Unused Docker Resources (Optional)**
If you also want to remove **unused build caches**, run:
```sh
docker system prune -a --volumes -f
```
- `-a` → Removes **all unused images**, including dangling and unreferenced ones.
- `--volumes` → Removes **unused volumes**.
- `-f` → Forces execution without confirmation.

---

### **Final Cleanup Check**
Run the following to confirm everything is removed:
```sh
docker ps -a     # Should return nothing (no containers)
docker images    # Should return nothing (no images)
docker volume ls # Should return nothing (no volumes)
docker network ls # Should only show default networks (bridge, host, none)
```

Now your **Docker environment is completely clean**! 
