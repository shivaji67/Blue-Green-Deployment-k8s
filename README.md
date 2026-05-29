# Blue-Green Deployment Using Jenkins and Kubernetes

## Overview

This project demonstrates Blue-Green Deployment using:

* Jenkins Freestyle Job
* Kubernetes
* GitHub Repository
* Service Selector Switching

The deployment process allows traffic to be shifted between Blue and Green environments without downtime.

---

## Project Structure

```text
Blue-Green-Deployment-k8s/
│
├── blue-deployment.yaml
├── green-deployment.yaml
├── service.yaml
└── README.md
```

---

## Prerequisites

* Kubernetes Cluster
* Jenkins Server
* Git Installed
* kubectl Installed
* GitHub Repository

Verify Kubernetes:

```bash
kubectl get nodes
```

---

## Kubernetes Access for Jenkins

Since Jenkins runs as a separate user, Kubernetes configuration must be copied to Jenkins.

Create kubeconfig directory:

```bash
sudo mkdir -p /var/lib/jenkins/.kube
```

Copy Kubernetes admin configuration:

```bash
sudo cp /etc/kubernetes/admin.conf \
/var/lib/jenkins/.kube/config
```

Assign ownership:

```bash
sudo chown -R jenkins:jenkins \
/var/lib/jenkins/.kube
```

Set permissions:

```bash
sudo chmod 600 \
/var/lib/jenkins/.kube/config
```

Verify:

```bash
sudo su - jenkins

kubectl get nodes
kubectl config current-context
```

Expected:

```text
kubernetes-admin@kubernetes
```

---

## Deployment Files

### Blue Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      env: blue
  template:
    metadata:
      labels:
        app: myapp
        env: blue
    spec:
      containers:
      - name: myapp
        image: nginx
        ports:
        - containerPort: 80
```

### Green Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      env: green
  template:
    metadata:
      labels:
        app: myapp
        env: green
    spec:
      containers:
      - name: myapp
        image: nginx
        ports:
        - containerPort: 80
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    env: blue
  ports:
  - port: 80
    targetPort: 80
```

---

## Jenkins Configuration

### Create Freestyle Job

Navigate to:

```text
Jenkins Dashboard → New Item → Freestyle Project
```

Project Name:

```text
Blue-Green-Deployment
```

---

### Source Code Management

Select:

```text
Git
```

Repository URL:

```text
https://github.com/<your-username>/Blue-Green-Deployment-k8s.git
```

Branch:

```text
*/main
```

---

### Build Parameter

Enable:

```text
This project is parameterized
```

Add:

```text
String Parameter
```

Configuration:

```text
Name: TARGET_ENV

Default Value: blue

Description: Enter blue or green
```

---

### Execute Shell

```bash
#!/bin/bash

export KUBECONFIG=/var/lib/jenkins/.kube/config

kubectl apply -f service.yaml

if [ "$TARGET_ENV" = "blue" ]; then

    kubectl apply -f blue-deployment.yaml

    kubectl set selector svc myapp-service env=blue

elif [ "$TARGET_ENV" = "green" ]; then

    kubectl apply -f green-deployment.yaml

    kubectl set selector svc myapp-service env=green

else

    echo "Invalid Environment"

    exit 1

fi

kubectl get svc myapp-service
kubectl get pods
```

---

## Deploy Blue Environment

Build with Parameters:

```text
TARGET_ENV=blue
```

Jenkins Actions:

```bash
kubectl apply -f blue-deployment.yaml

kubectl set selector svc myapp-service env=blue
```

Traffic routes to Blue pods.

---

## Deploy Green Environment

Build with Parameters:

```text
TARGET_ENV=green
```

Jenkins Actions:

```bash
kubectl apply -f green-deployment.yaml

kubectl set selector svc myapp-service env=green
```

Traffic routes to Green pods.

---

## Verification

Check deployments:

```bash
kubectl get deployments
```

Check pods:

```bash
kubectl get pods --show-labels
```

Check service:

```bash
kubectl get svc
```

Check selector:

```bash
kubectl describe svc myapp-service
```

Example:

```text
Selector: app=myapp,env=blue
```

or

```text
Selector: app=myapp,env=green
```

---

## Rollback

To rollback traffic to Blue:

```text
TARGET_ENV=blue
```

Jenkins executes:

```bash
kubectl set selector svc myapp-service env=blue
```

Traffic immediately switches back to Blue pods.

---

## Features

* Blue-Green Deployment
* Zero-Downtime Traffic Switching
* Jenkins Freestyle Job
* GitHub Integration
* Kubernetes Service Selector Switching
* Easy Rollback Mechanism
* Killercoda Kubernetes Compatible

---

## Author

Shivaji
