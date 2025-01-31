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

## Kustomize for Configuration Management

This project uses Kustomize for managing Kubernetes manifest variations across different environments (dev, staging, prod). Kustomize is a built-in configuration management tool in kubectl that allows you to customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.

Key features of Kustomize used in this project:

- **Base and Overlay Structure**: The `base` directory contains the common resources, while the `overlays` directory contains environment-specific variations. This allows for a clean separation of concerns.

- **Resource Customization**: Kustomize allows modifying resources defined in the base. For example, in the `prod` overlay, the replica count and resource limits are increased compared to the base.

- **Generated Resources**: Kustomize can generate resources, such as Secrets, from files. This is used to manage sensitive data like database passwords separately for each environment.

- **Patch and Strategic Merge**: Kustomize supports patching resources using JSON6902 patch and strategic merge patch. This is used to make environment-specific changes to resources, like updating the namespace name.

To apply the Kustomize configuration for a specific environment, use the `-k` flag with `kubectl apply`, pointing to the overlay directory:

```bash
# Apply base configuration
kubectl apply -k base/

# Apply the dev configuration
kubectl apply -k overlays/dev

# Apply the staging configuration
kubectl apply -k overlays/staging

# Apply the prod configuration
kubectl apply -k overlays/prod
```

This Kustomize setup demonstrates how to manage configurations for multiple environments while keeping the common base configuration reusable and maintaining a single source of truth.

## Resource Quotas and Scaling

Resource quotas and scaling parameters are defined per environment using Kustomize overlays. Each overlay (dev, staging, prod) has its own `quotas.yaml` file which defines the resource quotas for the 'user' and 'admin' components in that environment.

The quotas define limits on:
- CPU request and limit
- Memory request and limit 
- Maximum number of pods

These quotas ensure that each environment uses resources appropriately and prevent any one component from consuming too many resources.

The HorizontalPodAutoscaler (HPA) configurations are also defined per environment in the overlay's `kustomization.yaml` file. The HPA parameters control the minimum and maximum number of replicas for each component based on CPU and memory utilization.

Refer to the `quotas.yaml` and `kustomization.yaml` files in each overlay directory (`overlays/dev`, `overlays/staging`, `overlays/prod`) for the specific values for each environment.

## Environment Configuration

The project uses Kustomize overlays for environment-specific configurations (dev, staging, prod). Each environment has its own set of secrets and configurations managed through `.env` files.

### Environment Variables Files

#### MySQL Secret Configuration (`mysql-secret.env`)
```env
DB_HOSTNAME=db               # Database hostname
DB_PORT=3306                # MySQL port
DB_NAME=pythonlogin         # Database name
DB_USERNAME=root            # Database user
DB_PASSWORD=[env-specific]  # Different for each environment
```

#### Webapp Secret Configuration (`webapp-secret.env`)
```env
DB_HOSTNAME=[env].svc.cluster.local  # Internal k8s DNS
DB_PORT=3306                         # MySQL port
DB_NAME=pythonlogin                  # Database name
DB_USERNAME=root                     # Database user
DB_PASSWORD=[env-specific]           # Environment-specific password
SECRET_KEY=[env-specific]            # Unique key per environment
```

Environment-specific values:
- Dev: Lower resource limits, simplified setup
- Staging: Intermediate configuration for testing
- Production: Full resources, optimized for performance

## Scaling Configuration

HPA settings per environment:
- Dev: Scales at 85% utilization (conservative)
- Staging: Scales at 80% utilization
- Production: Scales at 70% utilization (aggressive)

## Learning Exercises

1. Explore RBAC configurations:
```bash
# Check service accounts
kubectl get sa -n dev

# Examine role bindings
kubectl describe rolebinding -n dev
```

2. Monitor HPA behavior:
```bash
# Watch HPA status
kubectl get hpa -n staging -w

# Check current metrics
kubectl top pods -n prod
```

3. Test scaling behavior:
```bash
# Generate load and watch scaling
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -n dev -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://usermgmt-user-service; done"
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
kubectl delete ns dev staging prod
```

## Next Steps for Learning

After understanding this project, you can:
1. Implement more complex RBAC scenarios
2. Explore custom metrics for HPA
3. Add monitoring and logging
4. Implement service mesh
5. Explore backup strategies
6. Learn about GitOps principles

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
