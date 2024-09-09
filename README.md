# wisecow-deployment

### Problem Statement 1: Containerization and Deployment of Wisecow Application on Kubernetes

#### **Step 1: Dockerization**
1. **Create a `Dockerfile`:**
   ```Dockerfile
   # Dockerfile
   FROM node:14
   
   # Create and set the working directory in the container
   WORKDIR /app

   # Copy package.json and package-lock.json to the container
   COPY package*.json ./

   # Install dependencies
   RUN npm install

   # Copy the rest of the application code to the container
   COPY . .

   # Expose the port the app runs on
   EXPOSE 3000

   # Command to run the application
   CMD ["npm", "start"]
   ```

2. **Build and run the Docker image locally to test it:**
   ```bash
   docker build -t wisecow-app .
   docker run -p 3000:3000 wisecow-app
   ```

---

#### **Step 2: Kubernetes Deployment**

1. **Create Kubernetes manifests:**

   - **`deployment.yaml`:**
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
             image: <your-docker-registry>/wisecow-app:latest
             ports:
             - containerPort: 3000
     ```

   - **`service.yaml`:**
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

2. **Apply the manifests to your Kubernetes cluster:**
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

---

#### **Step 3: CI/CD Integration**

1. **Set up GitHub Actions for CI/CD pipeline.**
   - Create a `.github/workflows/deploy.yml` file in your repository:
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
               tags: <your-docker-registry>/wisecow-app:latest

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

---

#### **Step 4: TLS Implementation**

1. **Generate TLS certificates:**
   ```bash
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt
   ```

2. **Create a Kubernetes secret for TLS:**
   ```bash
   kubectl create secret tls wisecow-tls --key tls.key --cert tls.crt
   ```

3. **Modify your Kubernetes `Ingress` or `Service` to use the TLS secret.**

---

### Problem Statement 2: Script-based Solutions

#### **Option 1: System Health Monitoring Script (Bash)**

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

This script monitors CPU, memory, and disk usage. If any of the metrics exceed predefined thresholds (80% in this case), it prints an alert to the console. It also lists the top 5 running processes by CPU usage.

#### **Option 2: Log File Analyzer (Python)**

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

This Python script analyzes a web server log file (e.g., Nginx or Apache) and provides a report of the number of 404 errors, the most requested pages, and the IP addresses with the most requests.

---
