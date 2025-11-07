# Docker-Demo — Hands-on Labs

A step-by-step set of labs to learn Docker the practical way: images, containers, networking, data persistence, multi-stage builds, Compose, and a full project.

## Guided Labs

### Guided Lab 01: Install Docker

**File:** [`01_Install_Docker.md`](./01_Install_Docker.md)
Install Docker Engine and verify your setup with basic sanity checks.

### Guided Lab 02: Serve a React App with Nginx

**File:** [`02_React_application_using_Nginx.md`](./02_React_application_using_Nginx.md)
Package a static React build and serve it via the official Nginx image.

### Guided Lab 03: Dockerizing a React Application

**File:** [`03_Dockerizing_React_Application.md`](./03_Dockerizing_React_Application.md)
Create a Dockerfile, build an image, run the container, and iterate locally.

### Guided Lab 04: Sharing Docker Images

**File:** [`04_Sharing_Docker_Image.md`](./04_Sharing_Docker_Image.md)
Tag, push, and pull images using a remote registry (Docker Hub or private).

### Guided Lab 05: Docker Networking (Bridge, Host, None + Custom)

**File:** [`05_Docker_Networking.md`](./05_Docker_Networking.md)
Understand container isolation, custom bridges, DNS by container name, and when to expose ports.

### Guided Lab 06: Volumes & Data Persistence

**File:** [`06_Docker_Volume_and_Data_Persistence.md`](./06_Docker_Volume_and_Data_Persistence.md)
Bind mounts vs named volumes; persist logs/data the right way.

### Guided Lab 07: Volume Project (Writer/Reader Pattern)

**File:** [`07_Docker_Volume_Project.md`](./07_Docker_Volume_Project.md)
Two containers share a named volume—one writes, the other reads.

### Guided Lab 08: Docker Compose (Multi-Service in One File)

**File:** [`08_Docker_Compose.md`](./08_Docker_Compose.md)
Define services, networks, and volumes; bring up/down a stack with one command.

### Guided Lab 09: Multi-Stage Images

**File:** [`09_Multi-stage_Images.md`](./09_Multi-stage_Images.md)
Slim production images, faster builds, smaller attack surface.

### Guided Lab 10: Basic Docker Commands (Cheatsheet)

**File:** [`10_Basic_Docker_Commands.md`](./10_Basic_Docker_Commands.md)
Day-to-day CLI you’ll actually use: build, run, exec, logs, prune, etc.

### Guided Lab 11: Cleanup & Reset

**File:** [`11_Clean_up.md`](./11_Clean_up.md)
Safe ways to stop/remove containers, networks, volumes, and images.

## Project

### Project: Docker Project (End-to-End)

**File:** [`12_Docker_Project.md`](./12_Docker_Project.md)
Bring everything together to containerize and run a real application using Docker/Compose, with networking and persistent storage.

---

## Quick Start

```bash
# Clone
git clone https://github.com/<your-org-or-user>/Docker-Demo.git
cd Docker-Demo

# Pick a lab and follow the steps in its .md file
# Example: Compose lab
docker compose version
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose down     # add -v to remove volumes too
```

**Prerequisites**

* [How Internet Work | DevOps Essentials | Internet Protocols & DNS | ChatGPT | AI](https://www.youtube.com/watch?v=AJlN3YMh97A&list=PLVOdqXbCs7bX88JeUZmK4fKTq2hJ5VS89&index=4)
* [Linux for DevOps](https://www.youtube.com/watch?v=U4j_RtDm2EU&list=PLVOdqXbCs7bX88JeUZmK4fKTq2hJ5VS89&index=9) 
* [Git & GitHub for DevOps](https://www.youtube.com/watch?v=6CpdUhLsBRY&list=PLVOdqXbCs7bX88JeUZmK4fKTq2hJ5VS89&index=11)

---

## DevOps Micro-Internship (FREE Program)

Hands-on, real-world practice with modern DevOps tools and workflows. Learn by doing—Docker, Compose, CI/CD, Terraform, Ansible, and more.

### All Assignments

[Your DevOps Master Plan](https://docs.google.com/spreadsheets/d/1Y3d0Wvil2wDRQK3ANS9WqkQYMviQb0fHHfftEiPwtXQ/edit?gid=0#gid=0)

Track every assignment in one place:
👉 **All Assignments Spreadsheet** — *[link here](https://docs.google.com/spreadsheets/d/1HnlenHEjytvLJMy84bBF-5B1RABaY_BjbfwCj-qnvHM/edit?gid=610294991#gid=610294991)*

### Register for the Next Batch

Ready to join the next cohort?
👉 **Register Here** — *[link here](https://forms.gle/PSGTcWDSvpUJXYGN6)*

### Support

* Read each lab’s `.md` instructions carefully
* Ask questions in the community: **Discord**: [https://discord.gg/CqWTeeNb6p](https://discord.gg/CqWTeeNb6p)

---

## Notes & Best Practices

* Prefer **named volumes** over bind mounts for portability
* Use **multi-stage builds** to keep images small
* Add **healthchecks** and **depends_on (service_healthy)** in Compose
* Keep secrets in **.env** (never commit them)
* Use `docker compose down -v` for a full teardown when resetting environments

