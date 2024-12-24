# Kubernetes User Management System

A comprehensive user management system deployed on Kubernetes, featuring separate user and admin interfaces with a MySQL backend. The system is designed with security, scalability, and reliability in mind.

## Architecture Overview

The system is divided into two main namespaces:
- `usermgm`: Contains the user-facing and admin-facing applications
- `database`: Houses the MySQL database and related components

### Key Components

1. **User Interface**
   - Deployment: `usermgmt-user`
   - Service: NodePort access on port 31281
   - Handles regular user interactions
   - Higher priority class for better resource allocation

2. **Admin Interface**
   - Deployment: `usermgmt-admin`
   - Service: NodePort access on port 31280
   - Manages administrative functions
   - Lower priority class for resource efficiency

3. **Database**
   - StatefulSet: MySQL 8.0
   - Persistent storage using NFS-CSI
   - Headless service for stable networking
   - Automated initialization with user schema

## Prerequisites

- Kubernetes cluster (version 1.20+)
- NFS server configured and accessible
- CSI driver support
- `kubectl` configured with cluster access

## Security Features

### Network Security
- Network policies restrict database access to the `usermgm` namespace
- Pod security policies enforced at namespace level
- All containers run as non-root
- Capability restrictions applied

### Data Security
- Secrets management for sensitive configuration
- Encrypted database passwords
- Secure communication between components
- Base64 encoded sensitive data

### Resource Management
- Namespace-level resource quotas
- Priority classes for workload management
- CPU and memory limits for all containers
- Guaranteed QoS classes where needed

## Installation

### 1. Create Namespaces and Resource Quotas
```bash
kubectl apply -f 00-namespaces-resourcesq.yaml
```

### 2. Configure Storage
```bash
kubectl apply -f 01-storage-class.yaml
```

### 3. Set Up Database
```bash
kubectl apply -f 02-UserMgmt-ConfigMap.yaml
kubectl apply -f 03-mysql-secret.yaml
kubectl apply -f 04-mysql-statefulset.yaml
kubectl apply -f 05-mysql-clusterip-service.yaml
```

### 4. Deploy Applications
```bash
kubectl apply -f 06-UserMgmt-Secret.yaml
kubectl apply -f 07-UserMgmtAdmin-Deployment.yaml
kubectl apply -f 08-UserMgmtAdmin-Service.yaml
kubectl apply -f 09-UserMgmtUser-Deployment.yaml
kubectl apply -f 10-UserMgmtUser-Service.yaml
```

### 5. Apply Network Policies
```bash
kubectl apply -f 11-networkpolicy-for-db.yaml
```

## Configuration Details

### Resource Quotas

#### Database Namespace
```yaml
requests.cpu: "1000m"
requests.memory: "2048Mi"
limits.cpu: "2000m"
limits.memory: "4Gi"
pods: "4"
```

#### User Management Namespace
User Workloads:
```yaml
requests.cpu: "2500m"
requests.memory: "640Mi"
limits.cpu: "5000m"
limits.memory: "1280Mi"
pods: "10"
```

Admin Workloads:
```yaml
requests.cpu: "500m"
requests.memory: "128Mi"
limits.cpu: "1000m"
limits.memory: "256Mi"
pods: "2"
```

### Storage Configuration

The system uses NFS storage with the following specifications:
```yaml
provisioner: nfs.csi.k8s.io
parameters:
  server: 192.168.4.36
  share: /var/lib/vz/nfs-export/
```

## Access Information

### User Interface
- NodePort: 31281
- Internal Service Port: 81
- Container Port: 5000

### Admin Interface
- NodePort: 31280
- Internal Service Port: 80
- Container Port: 5000

### Database
- Internal Service: mysql-headless.database.svc.cluster.local
- Port: 3306

## Monitoring and Health Checks

All components include:
- Liveness probes
- Readiness probes
- Resource monitoring
- Init container checks for dependencies

### Probe Configuration
```yaml
livenessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 5
  periodSeconds: 10
```

## Default Credentials

> ⚠️ **Important**: Change these credentials in your own environment

Admin user:
- Username: admin
- Password: myVerysecurepass531.
- Email: admin@example.com

## Troubleshooting

### Common Issues

1. **Database Connection Failures**
   - Verify network policy is applied
   - Check secret configuration
   - Ensure database pod is running

2. **Storage Issues**
   - Verify NFS server accessibility
   - Check PVC status
   - Validate storage class configuration

3. **Resource Constraints**
   - Monitor namespace quotas
   - Check pod resource usage
   - Verify priority class assignment

### Debugging Commands

```bash
# Check pod status
kubectl get pods -n usermgm
kubectl get pods -n database

# View logs
kubectl logs -n usermgm deployment/usermgmt-user
kubectl logs -n usermgm deployment/usermgmt-admin
kubectl logs -n database statefulset/mysql

# Check resources
kubectl describe quota -n usermgm
kubectl describe quota -n database
```

## Maintenance

### Backup Procedures

1. Database Backups
```bash
kubectl exec -n database mysql-0 -- mysqldump -u root -p${MYSQL_ROOT_PASSWORD} pythonlogin > backup.sql
```

### Scaling

The system can be scaled by adjusting:
- Deployment replicas for user/admin interfaces
- Resource quotas for namespaces
- Storage capacity for database

---
## Notes

- Ensure that the NFS server specified in the `01-storage-class.yaml` file is reachable.
- The base64 encoded values in the Secret file can be updated using the command:
  ```bash
  echo -n 'value' | base64
