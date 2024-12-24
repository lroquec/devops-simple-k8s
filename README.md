# Kubernetes Learning Project: User Management System

This project demonstrates various Kubernetes concepts by deploying a simple user management application. It's designed for learning purposes, showing how different Kubernetes components work together.

## Project Overview

A simple web application with:
- User interface for regular users
- Admin interface for administrators
- MySQL database backend

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

To use in your cluster:
1. Update the NFS server IP and path in the manifest
```yaml
parameters:
  server: YOUR_NFS_SERVER_IP
  share: YOUR_NFS_SHARE_PATH
```
2. Apply the configuration:
```bash
kubectl apply -f 01-storage-class.yaml
```

### 3. ConfigMaps for Database Initialization
File: `02-UserMgmt-ConfigMap.yaml`

Shows how to:
- Use ConfigMaps to store non-sensitive configuration
- Initialize a database with schema and default data
- Mount ConfigMap data as files in pods

### 4. Secrets Management
Files: `03-mysql-secret.yaml`, `06-UserMgmt-Secret.yaml`

Demonstrates:
- Storing sensitive information in Kubernetes Secrets
- Using base64 encoding for secret values
- Referencing secrets in pod configurations

To adapt for your use:
1. Generate your own base64 encoded values:
```bash
echo -n 'your-value' | base64
```
2. Replace the values in the secret manifests
3. Apply the secrets:
```bash
kubectl apply -f 03-mysql-secret.yaml
kubectl apply -f 06-UserMgmt-Secret.yaml
```

### 5. Stateful Applications
File: `04-mysql-statefulset.yaml`

Demonstrates:
- Using StatefulSets for stateful applications
- Configuring persistent storage with PVC templates
- Setting up health checks (liveness and readiness probes)
- Resource requests and limits
- Using init containers

Key learning points:
- Why StatefulSet is used instead of Deployment for databases
- How volume claims work with StatefulSets
- Importance of health checks
- Resource management at pod level

### 6. Service Types
Files: `05-mysql-clusterip-service.yaml`, `08-UserMgmtAdmin-Service.yaml`, `10-UserMgmtUser-Service.yaml`

Shows different types of Kubernetes services:
- Headless service for StatefulSet
- NodePort services for external access
- How services enable internal communication

### 7. Deployments
Files: `07-UserMgmtAdmin-Deployment.yaml`, `09-UserMgmtUser-Deployment.yaml`

Demonstrates:
- Basic deployment configuration
- Using environment variables from secrets
- Container security context
- Pod priority classes
- Init containers for dependency checking

### 8. Network Policies
File: `11-networkpolicy-for-db.yaml`

Shows how to:
- Restrict pod-to-pod communication
- Configure namespace-based network policies
- Allow specific ports and protocols

## How to Use This Project

1. Clone the repository
2. Modify the configurations as needed:
   - Update NFS server details in storage class
   - Change NodePort values if needed
   - Update secret values
   
3. Apply the manifests in order:
```bash
# Apply manifests in sequence
for f in *.yaml; do kubectl apply -f $f; done
```

4. Verify the deployment:
```bash
# Check namespaces
kubectl get ns

# Check pods in each namespace
kubectl get pods -n usermgm
kubectl get pods -n database

# Check services
kubectl get svc -n usermgm
kubectl get svc -n database
```

## Learning Exercises

1. Try scaling the deployments:
```bash
kubectl scale deployment usermgmt-user -n usermgm --replicas=3
```

2. Test resource quotas:
```bash
kubectl describe quota -n usermgm
```

3. Experiment with network policies:
```bash
kubectl get networkpolicies -n database
```

4. Examine pod logs:
```bash
kubectl logs -n usermgm deployment/usermgmt-user
```

## Understanding Pod Scheduling

The project uses priority classes to demonstrate pod scheduling:
- Higher priority for user-facing components
- Lower priority for admin components

Try creating pods when resources are constrained to see how priority affects scheduling.

## Clean Up

To remove all resources:
```bash
# Delete namespaces (this will delete all resources within them)
kubectl delete ns usermgm database
```

## Next Steps for Learning

After understanding this project, you can:
1. Add more complex network policies
2. Implement horizontal pod autoscaling
3. Add monitoring and logging
4. Explore Kubernetes Ingress
5. Learn about ConfigMap and Secret updates

Remember: This is a learning project focused on demonstrating Kubernetes concepts. Additional security and reliability features would be needed for production use.
