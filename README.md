# DevOps Livecoding: Ansible & Docker Project

## Introduction

**DevOps Livecoding** is a project that automates the deployment of applications using Ansible and Docker. This project aims to simplify the management of environments and containers through dedicated scripts.

---

## Testcontainers

**Testcontainers** is a Java library that enables lightweight, temporary instances of databases, Selenium web browsers, or other services in Docker containers for JUnit tests.

---

## Documentation of `main.yml`

This file defines a GitHub Actions workflow for Continuous Integration (CI).

### Trigger Conditions (on)
The workflow runs when:
- A push is made to the `main` or `dev` branches.
- A pull request is created.

### Main Job: `test-backend`
This job runs on an Ubuntu 22.04 environment.

#### Steps:
1. **Code Checkout**:
   Retrieves the source code using `actions/checkout@v2.5.0`.

2. **JDK 17 Setup**:
   Installs the JDK using `actions/setup-java@v3` (Amazon Corretto distribution).

3. **Build and Test**:
   Runs Maven commands to build and test the project:
   ```bash
   mvn clean verify
   ```

---

## Why Push Docker Images?

Pushing Docker images to a centralized registry allows for:
- Simplified version management.
- Consistent deployments across different environments.
- Seamless integration with CI/CD workflows.
- Scalability and independent updates of services in a microservices architecture.

---

## Ansible Inventory Configuration

The project uses an inventory file to define hosts and their connection variables.

### Inventory Structure
```plaintext
devops-livecoding/
└── ansible/
    └── inventories/
        └── setup.yml
```

### Example Content: `setup.yml`
```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: /path/to/private/key
  children:
    prod:
      hosts:
        your_server:
```

---

## Basic Ansible Commands

### Test Connectivity:
```bash
ansible all -i inventories/setup.yml -m ping
```

### List Inventory Hosts:
```bash
ansible-inventory -i inventories/setup.yml --list
```

### View Host Variables:
```bash
ansible-inventory -i inventories/setup.yml --host your_server
```

### Retrieve System Information:
```bash
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

---

## Roles and Deployment Documentation

The project uses several roles to orchestrate the deployment of the application.

### Main Playbook
```yaml
- hosts: your_server
  become: true
  roles:
    - cleanup_containers
    - install_docker
    - create_network
    - create_volumes
    - launch_database
    - launch_app
    - launch_proxy
```

### Roles

#### **1. cleanup_containers**
Cleans the environment by stopping and removing existing containers.

```yaml
- name: Stop all running containers
  shell: "docker stop $(docker ps -q)"
  ignore_errors: true

- name: Remove all containers
  shell: "docker rm $(docker ps -aq)"
  ignore_errors: true
```

#### **2. install_docker**
Installs Docker and its dependencies.

```yaml
- name: Install Docker
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - docker
    - python3-pip

- name: Install Docker SDK for Python
  ansible.builtin.pip:
    name: docker
```

#### **3. create_network**
Creates a Docker network for container communication.

```yaml
- name: Create Docker network
  community.docker.docker_network:
    name: my-network
    state: present
```

#### **4. create_volumes**
Creates volumes for persistent storage.

```yaml
- name: Create Docker volume for database
  community.docker.docker_volume:
    name: db-volume
    state: present
```

#### **5. launch_database**
Deploys the database container.

```yaml
- name: Run Database
  community.docker.docker_container:
    name: my-db
    image: foderac3/tp-devops-simple-api-database
    env:
      POSTGRES_DB: db
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pwd
    networks:
      - name: my-network
    volumes:
      - db-volume:/var/lib/postgresql/data
    state: started
```

#### **6. launch_app**
Deploys the backend application.

```yaml
- name: Run Backend Application
  community.docker.docker_container:
    name: my-api
    image: foderac3/tp-devops-simple-api-backend
    networks:
      - name: my-network
    env:
      DATABASE_HOST: my-db
      DATABASE_PORT: "5432"
    state: started
```

#### **7. launch_proxy**
Deploys an HTTP server to route requests.

```yaml
- name: Run HTTP Server
  community.docker.docker_container:
    name: httpd
    image: your_http_server_image
    ports:
      - "80:80"
    networks:
      - name: my-network
    state: started
```

---

## Usage

Run the playbook with the following command:
```bash
ansible-playbook -i inventories/setup.yml playbook.yml
```
