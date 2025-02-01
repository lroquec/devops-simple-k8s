# Kubernetes Learning Project: User Management System

This repository contains Kubernetes manifests that demonstrate various Kubernetes concepts through a practical user management application. It works in conjunction with:

- Application Code Repository: [devops-simple](https://github.com/lroquec/devops-simple)
  - Contains the application source code
  - Handles CI pipeline with GitHub Actions
  - Automatically updates image tags in this repository

This project is designed for learning purposes, helping you understand how different Kubernetes components work together in a real-world scenario.

## Project Overview

The project implements a simple web application consisting of:
- User interface for regular users
- Admin interface for administrators
- MySQL database backend
- Kustomize-based configuration management
- Multiple environment configurations (dev/staging/prod)

## Kubernetes Concepts Demonstrated

### 1. Namespaces and Resource Management
File: `00-namespaces-resourcesq.yaml`

Learn about:
- Namespace creation
- Pod security policies
- Priority classes for workload management
- Resource quotas configuration

### 2. Storage Configuration
File: `01-storage-class.yaml`

Understand:
- Storage class configuration
- NFS storage provider setup
- Storage provisioning and management
- PersistentVolume and PersistentVolumeClaim concepts

### 3. RBAC (Role-Based Access Control)
File: `02-rbac-and-sa.yaml`

Explore:
- Service Account creation
- Role and RoleBinding configuration
- Principle of least privilege
- Access control for metrics and resources

Key concepts demonstrated:
- Role-based security
- Service account management
- Permission scoping
- Security best practices

### 4. ConfigMaps and Database Configuration
Files: `03-mysql-configMap.yaml`, `04-mysql-statefulset.yaml`

Learn about:
- ConfigMap usage for configuration
- Database initialization
- StatefulSet deployment
- Persistent storage management

### 5. Application Deployment
Files: `06-UserMgmtAdmin-Deployment.yaml`, `08-UserMgmtUser-Deployment.yaml`

Understand:
- Deployment configuration
- Container security settings
- Resource limits and requests
- Health checks and probes
- Multi-container deployments

### 6. Network Configuration
Files: `05-mysql-clusterip-service.yaml`, `07-UserMgmtAdmin-Service.yaml`, `09-UserMgmtUser-Service.yaml`, `10-networkpolicy-for-db.yaml`

Study:
- Service types (ClusterIP, NodePort)
- Network policies
- Pod-to-pod communication
- External access configuration

### 7. Autoscaling
File: `11-hpa-config.yaml`

Learn about:
- Horizontal Pod Autoscaling
- Resource-based scaling
- Scaling thresholds and metrics
- Multi-metric scaling decisions

## Environment Management with Kustomize

This project uses Kustomize to demonstrate environment-specific configuration management. The structure shows:

```
base/           # Base configurations
overlays/
  ├── dev/     # Development environment
  ├── staging/ # Staging environment
  └── prod/    # Production environment
```

Each environment demonstrates different configurations for:
- Resource quotas
- Scaling parameters
- Replica counts
- Performance thresholds

## Prerequisites

- Kubernetes cluster 1.31+
- kubectl with Kustomize support
- Metrics server installed
- CNI with Network Policy support

## Installation and Testing

```bash
# Try different environments
kubectl apply -k overlays/dev
kubectl apply -k overlays/staging
kubectl apply -k overlays/prod

# Clean up
kubectl delete -k overlays/dev
```

### Learning Exercise: Environment Comparison

Compare configurations across environments:
```bash
# View dev quotas
kubectl get resourcequota -n dev

# Compare with production
kubectl get resourcequota -n prod

# Check scaling settings
kubectl get hpa -n staging
```

## Practice Exercises

1. **Explore RBAC:**
```bash
# List service accounts
kubectl get sa -n dev

# View role bindings
kubectl describe rolebinding -n dev
```

2. **Monitor Scaling:**
```bash
# Watch HPA in action
kubectl get hpa -n dev -w

# Generate test load
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://usermgmt-user-service; done"
```

3. **Study Network Policies:**
```bash
# Examine network policies
kubectl describe networkpolicy -n dev

# Test connectivity
kubectl run test-pod --rm -it --image=busybox -- /bin/sh
```

# Continuous Integration and Deployment

This project demonstrates a complete CI/CD pipeline using GitHub Actions and ArgoCD.

## GitHub Actions Integration

The container images used in this project (`usermgm-admin` and `usermgm-user`) are automatically built and updated through a GitHub Actions workflow in the application code repository: [devops-simple](https://github.com/lroquec/devops-simple)

The workflow:
1. Builds new container images when changes are pushed
2. Updates the image tags in this Kubernetes manifests repository
3. Ensures our deployment always uses the latest version of the application

## ArgoCD Deployment

The project is deployed and managed using ArgoCD, demonstrating GitOps principles:

- ArgoCD monitors this repository for changes
- When manifests are updated (either manually or via GitHub Actions):
  - ArgoCD automatically detects the changes
  - Applies the new configurations to the cluster
  - Ensures the desired state matches the actual state

This setup demonstrates:
- GitOps workflow in practice
- Automated deployment pipeline
- Infrastructure as Code principles
- Continuous deployment capabilities

Remember: This is a learning project focused on demonstrating Kubernetes concepts, CI/CD practices, and GitOps principles. Additional security and reliability features would be needed for production use.
