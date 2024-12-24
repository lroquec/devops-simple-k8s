# Kubernetes Learning Project: User Management System

This repository contains the Kubernetes manifests for deploying a user management application. It works in conjunction with:

- Application Code Repository: [devops-simple](https://github.com/lroquec/devops-simple)
  - Contains the application source code
  - Handles CI pipeline with GitHub Actions
  - Automatically updates image tags in this repository

This project demonstrates various Kubernetes concepts by deploying a simple user management application. It's designed for learning purposes, showing how different Kubernetes components work together.

## Project Overview

A simple web application with:
- User interface for regular users
- Admin interface for administrators
- MySQL database backend
- Automated CI/CD pipeline using GitHub Actions
- GitOps deployment using ArgoCD

## Kubernetes Concepts Demonstrated

### 1. Namespaces and Resource Management
File: `00-namespaces-resourcesq.yaml`

This manifest shows how to:
- Create separate namespaces for different application components
- Configure resource quotas to limit resource usage per namespace
- Set up priority classes for different types of workloads

```bash
# Create namespaces and quotas
kubectl apply -f 00-namespaces-resourcesq.yaml
```

### 2. Storage Configuration
File: `01-storage-class.yaml`

Demonstrates:
- How to configure a StorageClass for persistent storage
- Using NFS as a storage provider
- Setting up storage parameters and reclaim policies

### 3. Service Accounts and RBAC
File: `02-rbac-and-sa.yaml`

Demonstrates:
- Creating dedicated service accounts for each component
- Implementing RBAC for minimum required permissions
- Setting up roles for metrics access (required for HPA)
- Binding roles to service accounts

Key concepts:
- Principle of least privilege
- Role-based access control
- Security best practices

### 4. ConfigMaps and Secrets
Files: `03-UserMgmt-ConfigMap.yaml`, `04-mysql-secret.yaml`, `05-UserMgmt-Secret.yaml`

Shows how to:
- Use ConfigMaps for database initialization
- Manage sensitive information in Secrets
- Reference configurations in deployments

### 5. Stateful Applications
File: `06-mysql-statefulset.yaml`

Demonstrates:
- Using StatefulSets for databases
- Configuring persistent storage
- Setting up health checks
- Resource management
- Service account integration

### 6. Services and Networking
Files: `07-mysql-clusterip-service.yaml` through `11-UserMgmtUser-Service.yaml`

Shows different types of Kubernetes services and networking concepts

### 7. Network Policies
File: `12-networkpolicy-for-db.yaml`

Shows how to:
- Restrict pod-to-pod communication
- Configure namespace-based policies
- Implement security at network level

### 8. Horizontal Pod Autoscaling
File: `13-hpa-config.yaml`

Demonstrates:
- Automatic scaling based on resource utilization
- Different scaling configurations for admin and user interfaces
- Integration with resource quotas
- Metrics-based scaling decisions

## Resource Quotas and Scaling

### Database Namespace Quotas
- CPU Request: 1000m
- CPU Limit: 2000m
- Memory Request: 2048Mi
- Memory Limit: 4Gi
- Max Pods: 4

### Admin UI Quotas and Scaling
- Resource Quotas:
  - CPU Request: 500m
  - CPU Limit: 1000m
  - Memory Request: 128Mi
  - Memory Limit: 256Mi
  - Max Pods: 2
- HPA Configuration:
  - Min Replicas: 1
  - Max Replicas: 2
  - CPU Target: 80%
  - Memory Target: 80%

### User UI Quotas and Scaling
- Resource Quotas:
  - CPU Request: 2500m
  - CPU Limit: 5000m
  - Memory Request: 640Mi
  - Memory Limit: 1280Mi
  - Max Pods: 10
- HPA Configuration:
  - Min Replicas: 1
  - Max Replicas: 5
  - CPU Target: 80%
  - Memory Target: 80%

## Learning Exercises

1. Explore RBAC configurations:
```bash
# Check service accounts
kubectl get sa -n usermgm

# Examine role bindings
kubectl describe rolebinding -n usermgm
```

2. Monitor HPA behavior:
```bash
# Watch HPA status
kubectl get hpa -n usermgm -w

# Check current metrics
kubectl top pods -n usermgm
```

3. Test scaling behavior:
```bash
# Generate load and watch scaling
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://usermgmt-user-service; done"
```

## Prerequisites

- Kubernetes cluster 1.31+
- Metrics Server installed (for HPA)
- NFS Server configured
- CNI with Network Policy support. Tested with calico.

## Clean Up

To remove all resources:
```bash
# Delete namespaces (this will delete all resources within them)
kubectl delete ns usermgm database
```

## Next Steps for Learning

After understanding this project, you can:
1. Implement more complex RBAC scenarios
2. Explore custom metrics for HPA
3. Add monitoring and logging
4. Implement service mesh
5. Explore backup strategies
6. Learn about GitOps principles

## Understanding Pod Scheduling and Scaling

The project demonstrates advanced scheduling and scaling through:
- Priority classes for different workloads
- HPA for automatic scaling
- Resource quotas for capacity management
- RBAC for access control

This combination shows how Kubernetes manages:
- Resource allocation
- Workload priorities
- Automatic scaling
- Security boundaries

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
