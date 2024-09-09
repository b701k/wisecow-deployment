# wisecow-deployment

### **Problem 1: Containerization and Deployment of Wisecow Application on Kubernetes**

You will need the following files in your repository:

1. **Dockerfile**
2. **Kubernetes YAML Files**
   - `deployment.yaml`
   - `service.yaml`
3. **GitHub Actions Workflow**
   - `.github/workflows/deploy.yml`
4. **README.md**
   - Documenting your project.

### **Problem 2: Scripts for System Health Monitoring and Log File Analyzer**

You will need the following files in your repository:

1. **system_health_monitor.sh** (Bash Script for System Health Monitoring)
2. **log_file_analyzer.py** (Python Script for Log File Analysis)
3. **README.md**
   - Documenting your scripts.

---

#### **Step-by-Step GitHub Structure**

Below, I've listed the files and their contents so that you can create them in your local directory. Then, I will guide you on how to push them to GitHub.

---

### **Problem 1: Project Structure**

1. **Dockerfile**
    ```dockerfile
    # Dockerfile
    FROM node:14
    
    WORKDIR /app

    COPY package*.json ./
    RUN npm install

    COPY . .

    EXPOSE 3000
    CMD ["npm", "start"]
    ```

2. **Kubernetes Deployment and Service**

    - **deployment.yaml**
      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: wisecow-deployment
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: wisecow
        template:
          metadata:
            labels:
              app: wisecow
          spec:
            containers:
            - name: wisecow-container
              image: your-docker-repo/wisecow-app:latest
              ports:
              - containerPort: 3000
      ```

    - **service.yaml**
      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: wisecow-service
      spec:
        selector:
          app: wisecow
        ports:
          - protocol: TCP
            port: 80
            targetPort: 3000
        type: LoadBalancer
      ```

3. **GitHub Actions Workflow**

   - **.github/workflows/deploy.yml**
     ```yaml
     name: CI/CD Pipeline

     on:
       push:
         branches:
           - main

     jobs:
       build:
         runs-on: ubuntu-latest
         steps:
           - name: Checkout code
             uses: actions/checkout@v2

           - name: Set up Docker Buildx
             uses: docker/setup-buildx-action@v1

           - name: Build and push Docker image
             uses: docker/build-push-action@v2
             with:
               push: true
               tags: your-docker-repo/wisecow-app:latest

       deploy:
         runs-on: ubuntu-latest
         needs: build
         steps:
           - name: Set up kubectl
             uses: azure/setup-kubectl@v1
             with:
               version: 'v1.18.0'

           - name: Deploy to Kubernetes
             run: |
               kubectl apply -f deployment.yaml
               kubectl apply -f service.yaml
     ```

4. **README.md**
    ```markdown
    # Wisecow Application Deployment on Kubernetes

    This repository contains the necessary files for containerizing and deploying the Wisecow application on Kubernetes with a CI/CD pipeline using GitHub Actions.

    ## Files
    - Dockerfile: Used for containerizing the application.
    - deployment.yaml: Kubernetes deployment manifest.
    - service.yaml: Kubernetes service manifest.
    - .github/workflows/deploy.yml: GitHub Actions CI/CD pipeline for automating deployment.

    ## How to Use
    1. Build and run the Docker container locally:
       ```bash
       docker build -t wisecow-app .
       docker run -p 3000:3000 wisecow-app
       ```

    2. Apply Kubernetes manifests:
       ```bash
       kubectl apply -f deployment.yaml
       kubectl apply -f service.yaml
       ```

    3. CI/CD Pipeline:
       - Push to the main branch triggers the build and deployment process.
    ```

---

### **Problem 2: Script Solutions**

1. **system_health_monitor.sh**
    ```bash
    #!/bin/bash

    # Thresholds
    CPU_THRESHOLD=80
    MEM_THRESHOLD=80
    DISK_THRESHOLD=80

    # CPU Usage
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
    if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
        echo "CPU usage is above threshold: $CPU_USAGE%"
    fi

    # Memory Usage
    MEM_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
    if (( $(echo "$MEM_USAGE > $MEM_THRESHOLD" | bc -l) )); then
        echo "Memory usage is above threshold: $MEM_USAGE%"
    fi

    # Disk Usage
    DISK_USAGE=$(df -h / | grep / | awk '{ print $5 }' | sed 's/%//g')
    if [ $DISK_USAGE -gt $DISK_THRESHOLD ]; then
        echo "Disk usage is above threshold: $DISK_USAGE%"
    fi

    # Running Processes
    echo "Top 5 Running Processes:"
    ps aux --sort=-%cpu | head -n 6
    ```

2. **log_file_analyzer.py**
    ```python
    import re
    from collections import Counter

    # Load log file
    log_file = '/path/to/your/webserver/access.log'
    with open(log_file) as f:
        log_data = f.readlines()

    # Analyze for 404 errors
    status_codes = [re.search(r'\" (\d{3}) ', line).group(1) for line in log_data if re.search(r'\" (\d{3}) ', line)]
    not_found_errors = [code for code in status_codes if code == '404']
    print(f"Number of 404 errors: {len(not_found_errors)}")

    # Most requested pages
    pages = [re.search(r'\"(GET|POST) (.*?) ', line).group(2) for line in log_data if re.search(r'\"(GET|POST) (.*?) ', line)]
    most_requested = Counter(pages).most_common(5)
    print("Most requested pages:")
    for page, count in most_requested:
        print(f"{page}: {count} requests")

    # Top IP addresses
    ips = [re.search(r'^(\S+)', line).group(1) for line in log_data]
    most_common_ips = Counter(ips).most_common(5)
    print("Top IP addresses:")
    for ip, count in most_common_ips:
        print(f"{ip}: {count} requests")
    ```

3. **README.md**
    ```markdown
    # Script Solutions for System Monitoring and Log Analysis

    This repository contains two scripts that address the following tasks:

    1. **System Health Monitoring Script** (Bash)
       - Monitors CPU, memory, and disk usage.
       - Alerts if any thresholds are exceeded.

    2. **Log File Analyzer** (Python)
       - Analyzes web server logs for common patterns such as 404 errors, most requested pages, and top IP addresses.

    ## How to Use

    1. **System Health Monitoring Script:**
       - Run the script in a terminal:
         ```bash
         ./system_health_monitor.sh
         ```

    2. **Log File Analyzer:**
       - Run the Python script:
         ```bash
         python3 log_file_analyzer.py
         ```
       - Ensure to set the correct path to your log file in the script.
    ```

---

### **Final Steps: Upload to GitHub**

1. **Set Up Your Directory Structure Locally:**
   - Create the folders and files as described above.
   - Ensure the `.github/workflows/deploy.yml` path is correct.

2. **Initialize Git and Push to GitHub:**
   - Navigate to your project folder and run the following commands:
     ```bash
     git init
     git add .
     git commit -m "Initial commit for Wisecow deployment and script solutions"
     git remote add origin https://github.com/your-username/repository-name.git
     git push -u origin main
     ```

3. **Verify on GitHub:**
   - After pushing, go to your GitHub repository to verify that all the files are uploaded correctly.

This setup should give you everything you need to submit the assignment. Let me know if you need further assistance!
