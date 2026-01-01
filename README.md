# Kind Kubernetes Cluster Setup on AWS EC2 (Ubuntu)

This repository contains a step-by-step guide to install and configure a **kind Kubernetes cluster** on an Ubuntu EC2 instance, deploy an `nginx` application, and access it externally.

## Table of Contents

* [Prerequisites](#prerequisites)
* [Install Dependencies](#install-dependencies)
* [Create Kind Cluster](#create-kind-cluster)
* [Deploy Nginx](#deploy-nginx)
* [Access Nginx](#access-nginx)
* [Cleanup](#cleanup)
* [Notes](#notes)
* [References](#references)

## Prerequisites

* Ubuntu EC2 instance (t2.medium recommended)
* Security group: TCP 22 (SSH) and 8080 (nginx access) allowed
* Basic Linux command knowledge

## Install Dependencies

### Update system

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim
```

### Docker

```bash
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
```

> Logout & login again for Docker group changes to take effect.

Verify:

```bash
docker version
```

### kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

## Create Kind Cluster

### kind YAML configuration (`kind-t2-medium.yaml`)

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 80
        hostPort: 8080
  - role: worker
  - role: worker
```

### Create cluster

```bash
kind create cluster --name t2-medium-cluster --config kind-t2-medium.yaml
kubectl cluster-info --context kind-t2-medium-cluster
```

## Deploy Nginx

### Deployment YAML (`nginx-deploy.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      nodeSelector:
        kubernetes.io/role: control-plane
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### Apply deployment

```bash
kubectl apply -f nginx-deploy.yaml
kubectl get pods -o wide
kubectl expose deployment nginx --port=80 --target-port=80
kubectl get svc -o wide
```

## Access Nginx

Since kind runs in Docker, use port-forward:

```bash
kubectl port-forward --address 0.0.0.0 deployment/nginx 8080:80
```

Open in browser or test with curl:

```bash
curl http://<EC2_PUBLIC_IP>:8080
```

## Cleanup

```bash
kind delete cluster --name t2-medium-cluster
```

## Notes

* Ensure EC2 security group allows TCP 8080
* This setup is for testing/learning only
* For production external access, consider MetalLB or a cloud Kubernetes service

## References

* [Kind Documentation](https://kind.sigs.k8s.io/docs/user/quick-start/)
* [Kubernetes Official Docs](https://kubernetes.io/docs/home/)
* [MetalLB](https://metallb.universe.tf/)

## Prepared by:
* Shaik.Moulali
* Cloud & DevOps Engineer
* moula.cloud5@gmail.com
* [LinkedIn](www.linkedin.com/in/shaik-moulali-517b35278)
