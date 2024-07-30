## Project Problem Statement
A development team needs to establish a basic CI/CD pipeline for a web application. The goal is to automate version control, containerization, building, testing, and deployment processes.
### Deliverables
### 1.Git Repository:
+ Create a Git repository: Initialize a new repository for the web application.
+ Branching Strategy:
 Set up main and develop branches.
 Create a feature branch for a new feature or bug fix.
+ Add Configuration Files:
 Create a .gitignore file to exclude files like logs, temporary files, etc.
 Create a README.md file with a project description, setup instructions, and contribution guidelines.
 ![alt text](<image/Screenshot from 2024-07-29 16-16-46.png>)
 ![alt text](<image/Screenshot from 2024-07-29 16-22-42.png>)
## 2.Docker Configuration:
+ Dockerfile:
Write a Dockerfile to define how to build the Docker image for the web application.
```bash
FROM nginx:1.10.1-alpine
COPY index.html /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```
+ Docker Ignore File:
Create a .dockerignore file to exclude files and directories from the Docker build context.
+ Image Management:
Build a Docker image using the Dockerfile.
```bash
docker build -t santoshpagire/mycustomnginx  .
```
Push the built Docker image to a container registry (e.g., Docker Hub).
```bash
docker push santoshpagire/mycustomnginx
```
![alt text](<image/Screenshot from 2024-07-30 07-46-32.png>)
## 3.Jenkins Configuration:
+ Jenkins Job Setup:
+ Create a Jenkins job to pull code from the Git repository.
![alt text](<image/Screenshot from 2024-07-29 22-38-42.png>)
+ Configure Jenkins to build the Docker image using the Dockerfile.
![alt text](<image/Screenshot from 2024-07-29 22-39-13.png>)
+ Configure Jenkins to push the Docker image to the container registry after a successful build.
![alt text](<image/Screenshot from 2024-07-29 22-39-39.png>)
Jenkins Pipeline:
+ Create a Jenkinsfile to define the CI/CD pipeline stages, including build, test, and deploy.
> Jenkinsfile
```bash
pipeline {
    agent any
    environment {
        ANSIBLE_SUDO_PASSWORD = credentials('Ansible')
        registry = 'docker.io'  
        registryCredential = 'docker' 
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/santoshPagire/Task_Day15.git', branch: 'main'
            }
        }

        stage('build image') {
            steps{
                script{
                    docker.withRegistry('', registryCredential){
                        def customImage = docker.build("santoshpagire/mynginx-app:latest")
                        customImage.push()
                       


                    }

                }
            }
        }
        stage('Run Playbook') {
            steps{
               sh """
                   export ANSIBLE_SUDO_PASSWORD=${ANSIBLE_SUDO_PASSWORD}
                   ansible-playbook playbook.yml -i inventory -u jenkins --extra-vars "ansible_become_pass=${ANSIBLE_SUDO_PASSWORD}"
                """
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }

}
 
```

## 4.Ansible Playbook:
+ Basic Playbook Creation:
Develop an Ansible playbook to automate the deployment of the Docker container.
+ Playbook Tasks:
Install Docker on the target server (if Docker is not already installed).
Pull the Docker image from the container registry.
Run the Docker container with the required configurations.
> playbook.yml
```bash
---
- hosts: target1
  become: true
  tasks:
    - name: Install dependencies
      apt:
        name: "{{ item }}"
        state: present
      with_items:
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg
        - lsb-release
        - python3
        - python3-pip
    - name: Install Docker SDK for Python
      pip:
        name: "docker<5"
    
    - name: Setup more docker dependencies
      pip:
        name: "websocket-client<1" 

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable
        state: present

    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    - name: Add user to Docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes

    - name: Start Docker service
      service:
        name: docker
        state: started

    - name: Removing sudo from commands
      command:
        cmd: usermod -aG docker {{ansible_user}}
    
    - name: pull an image
      docker_image:
        name: santoshpagire/customnginx-app
        tag: latest 
        source: pull
    
    - name: Create a data container
      docker_container:
        name: myApp
        image: santoshpagire/customnginx-app
        state: started
        restart_policy: always
        ports:
          - "81:80"
```
![alt text](<image/Screenshot from 2024-07-29 22-40-53.png>)
![alt text](<image/Screenshot from 2024-07-30 07-42-22.png>)
+ Inventory File:
Create an inventory file specifying the target server(s) for deployment.
```bash
target ansible_host=170.00.00.00 ansible_connection=ssh ansible_user=User ansible_ssh_pass=User_Pass

```
