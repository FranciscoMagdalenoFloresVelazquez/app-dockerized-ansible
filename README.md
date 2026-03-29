```markdown
# Full Project: Complete CI/CD & Deployment with Ansible

This project implements a fully automated DevOps pipeline for a simple Python application. It uses a modern CI/CD approach to build, test, and deploy a Dockerized application using GitHub Actions and Ansible Roles.

## 🚀 Overview

The workflow is divided into two main parts:

1.  **Continuous Integration (CI)**: A GitHub Actions pipeline builds the application image, runs tests, and pushes the finalized image to Docker Hub.
2.  **Continuous Deployment (CD)**: An Ansible playbook is triggered from the CI pipeline to automatically pull the new Docker image and deploy it to a remote server. The remote server is pre-configured with common packages, Docker, and an NPM container for expose the application.

## 📁 Project Structure

```text
.
├── ansible/
│   ├── inventory/
│   │   ├── group_vars/
│   │   │   └── all.yaml          # Global variables (e.g., dev_user)
│   │   └── hosts.ini             # Target server inventory
│   ├── playbooks/
│   │   └── site.yaml             # Main site deployment playbook
│   ├── roles/                    # Modular Ansible configurations
│   │   ├── app/                  # Application deployment logic
│   │   ├── common/               # Basic system configuration
│   │   ├── docker/               # Docker engine installation
│   │   └── npm/                  # Nginx Proxy Manager setup
│   ├── ansible.cfg               # Ansible configuration file
│   └── requirements.yaml         # Community collections (e.g., community.docker)
├── app/                          # The core application
│   ├── Dockerfile                # Containerization instructions
│   ├── app.py                    # Simple Python web application
│   └── requirements.txt          # Python dependencies
└── README.md                     # This documentation
```

## 🛠️ Ansible Key Components

This project uses an organized Ansible structure:

### ⚙️ Ansible Configuration (`ansible.cfg`)
Contains project-wide settings, such as inventory paths and specific role paths to optimize the development environment.

### 📦 Requirements (`requirements.yaml`)
We use the `community.docker` collection to manage Docker containers efficiently. This file ensures all required external collections are installed.

### 🌍 Global Variables (`group_vars/all.yaml`)
Defines variables available to all roles. For example, `dev_user` is defined here to ensure consistent deployment permissions across the server.

### 🧱 Ansible Roles

* **`common`**: Updates system packages and installs essential tools.
* **`docker`**: Installs and configures the Docker engine.
* **`npm`**: Sets up Nginx Proxy Manager (NPM) in a Docker container, providing an easy way to manage SSL certificates and reverse proxies for public internet access.
* **`app`**: Deploys the Python application using the Docker image built in the pipeline.

## 🔄 CI/CD Pipeline

The project uses GitHub Actions to automate the entire lifecycle.

### CI: Build & Test Stage

1.  **Lint/Test**: Analyzes the Python code and runs unit tests.
2.  **Docker Build**: Builds the Docker image from `app/Dockerfile`.
3.  **Security Scan**: Scans the image for vulnerabilities.
4.  **Docker Push**: Tags and pushes the image to Docker Hub.

### CD: Automated Deployment Stage

This stage runs an Ansible playbook on the self-hosted GitHub Actions runner.

1.  **Ansible Installation**: Installs Ansible on the runner environment.
2.  **Inventory Creation**: Populates the `hosts.ini` with deployment secrets (IP, keys).
3.  **Community Collection Setup**: Installs `community.docker` from `requirements.yaml`.
4.  **Playbook Execution**: Runs `playbooks/site.yaml`.
5.  **Role Orchestration**: The playbook executes roles sequentially:
    * Prepares the server (`common`).
    * Installs Docker (`docker`).
    * Deploys NPM (`npm`).
    * Deploys the application with the new image (`app`).

## ⚙️ How to Run Locally (Manual Deployment)

To manually trigger a deployment from your local machine, follow these steps.

### Prerequisites

* Python 3 & Ansible installed.
* A target server (e.g., a virtual machine or VPS) accessible via SSH.

### Step 1: Install Ansible Dependencies

Run this command in the project root:

```bash
ansible-galaxy install -r ansible/requirements.yaml
```

### Step 2: Configure Inventory

Edit `ansible/inventory/hosts.ini` and add your server's IP and connection details:

```ini
[all]
your_server_ip_or_hostname ansible_user=root ansible_ssh_private_key_file=/path/to/your/key
```

### Step 3: Configure Global Variables

Check `ansible/inventory/group_vars/all.yaml` and set your preferred `dev_user`.

### Step 4: Run the Deployment Playbook

From the `ansible` directory:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/site.yaml
```

## 🌐 Expose to Internet

Once deployed, your application is accessible locally. To expose it to the internet:

1.  **Port Forwarding (OpenWrt/Router)**: Configure port forwarding on your router/gateway to point incoming TCP traffic on ports 5954 and 5956 to the local IP of your server running NPM.
2.  **Nginx Proxy Manager**: Access the NPM administration console (usually port 5955).
3.  **Proxy Host**: Create a new "Proxy Host" entry in NPM. Set your domain name and point it to the container name or local IP of your application.
4.  **SSL**: Use the NPM interface to automatically generate a free Let's Encrypt SSL certificate for secure access.

---
**Disclaimer**: This project is provided as-is for educational purposes. Security best practices (like firewall configuration and secret management) should be followed in a production environment.
```
