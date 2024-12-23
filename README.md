# User Management System Kubernetes Deployment

This repository contains Kubernetes manifests for deploying a user management system. The system includes two main applications (`usermgmt-admin` and `usermgmt-user`), along with a MySQL database for persistence. The setup ensures secure communication and scalable deployments using Kubernetes features.

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Components](#components)
4. [Prerequisites](#prerequisites)
5. [Deployment Steps](#deployment-steps)
6. [Key Configuration Files](#key-configuration-files)
7. [Security Considerations](#security-considerations)
8. [Troubleshooting](#troubleshooting)
9. [License](#license)

## Overview

This project provides a deployment setup for a user management system leveraging Kubernetes. The system includes:
- Admin and user-facing services
- MySQL as the database backend
- NFS-backed persistent storage

## Architecture

The deployment follows a modular architecture:
- **Namespaces**: Separate namespaces for application (`usermgm`) and database (`database`) to isolate workloads.
- **StatefulSet**: Ensures stable and persistent MySQL deployment.
- **Secrets**: Secure storage of database credentials and app secret for session management
- **Network Policies**: Restrict communication to authorized pods.
- **Storage Class**: NFS-based storage for persistent data.

## Key Configuration Files

- **[00-namespace.yaml](00-namespace.yaml)**: Defines namespaces for the project.
- **[01-storage-class.yaml](01-storage-class.yaml)**: Configures NFS-based storage.
- **[02-UserMgmt-ConfigMap.yaml](02-UserMgmt-ConfigMap.yaml)**: Database initialization script.
- **[03-mysql-secret.yaml](03-mysql-secret.yaml)**: Database credentials (base64 encoded).
- **[04-mysql-statefulset.yaml](04-mysql-statefulset.yaml)**: MySQL deployment as a StatefulSet.
- **[05-mysql-clusterip-service.yaml](05-mysql-clusterip-service.yaml)**: Headless service for MySQL.
- **[06-UserMgmt-Secret.yaml](06-UserMgmt-Secret.yaml)**: Application secrets.
- **[07-UserMgmtAdmin-Deployment.yaml](07-UserMgmtAdmin-Deployment.yaml)**: Deployment configuration for `usermgmt-admin`.
- **[08-UserMgmtAdmin-Service.yaml](08-UserMgmtAdmin-Service.yaml)**: Service for `usermgmt-admin`.
- **[09-UserMgmtUser-Deployment.yaml](09-UserMgmtUser-Deployment.yaml)**: Deployment configuration for `usermgmt-user`.
- **[10-UserMgmtUser-Service.yaml](10-UserMgmtUser-Service.yaml)**: Service for `usermgmt-user`.
- **[11-networkpolicy-for-db.yaml](11-networkpolicy-for-db.yaml)**: Network policy for database access.

## Prerequisites

- Kubernetes cluster (v1.20+ recommended)
- NFS server for persistent storage
- Access to Docker images for `usermgmt-admin` and `usermgmt-user`

## Deployment Steps

0. **Create the namespace resources**:
   ```bash
   kubectl apply -f 00-namespace.yaml
   ```

1. **Create the Storage Class**:
   ```bash
   kubectl apply -f 01-storage-class.yaml
   ```
2. **Create the ConfigMap**:
   ```bash
   kubectl apply -f 02-UserMgmt-ConfigMap.yaml
   ```
3. **Deploy the MySQL Server Secret**:
   ```bash
   kubectl apply -f 03-mysql-secret.yaml
   ```
4. **Deploy the MySQL Server**:
   ```bash
   kubectl apply -f 04-mysql-statefulset.yaml
   ```
5. **Expose MySQL Server**:
   ```bash
   kubectl apply -f 05-mysql-clusterip-service.yaml
   ```
6. **Create the Secret for app**:
   ```bash
   kubectl apply -f 06-UserMgmt-Secret.yaml
   ```
7. **Deploy and Expose the Admin Component**:
   ```bash
   kubectl apply -f 07-UserMgmtAdmin-Deployment.yaml
   kubectl apply -f 08-UserMgmtAdmin-Service.yaml
   ```
8. **Deploy and Expose the User Component**:
   ```bash
   kubectl apply -f 09-UserMgmtUser-Deployment.yaml
   kubectl apply -f 10-UserMgmtUser-Service.yaml
   ```
9. **Deploy the Network Policy Component**:
   ```bash
   kubectl apply -f 11-networkpolicy-for-db.yaml
   ```
## Configuration Details

### Environment Variables
Environment variables for both Admin and User components are sourced from `06-UserMgmt-Secret.yaml`. These include:
- `DB_HOSTNAME`
- `DB_PORT`
- `DB_NAME`
- `DB_USERNAME`
- `DB_PASSWORD`
- `SECRET_KEY`

### Database Initialization
The database schema and initial admin user are created using the SQL script defined in `03-UserMgmt-ConfigMap.yaml`.

### Resource Allocation
Each deployment defines resource requests and limits:
- **Requests**: 
  - Memory: `64Mi`
  - CPU: `250m`
- **Limits**: 
  - Memory: `128Mi`
  - CPU: `500m`

---

## Testing the Application

1. **Access the Admin Interface**:
   Navigate to the Node's IP address and the port specified in `usermgmt-admin-service` (default: 31280).

2. **Access the User Interface**:
   Navigate to the Node's IP address and the port specified in `usermgmt-user-service` (default: 31281).

---
## Troubleshooting

- Verify the status of all pods:
  ```bash
  kubectl get pods -n usermgm
  kubectl get pods -n database
  ```
- Verify environment variables
  ```bash
  kubectl exec -it -n usermgm usermgmt-user-<hash> -- python -c "import os; print(os.getenv('MYSQL_HOST'))"
  ```

---
## Notes

- Ensure that the NFS server specified in the `01-storage-class.yaml` file is reachable.
- The base64 encoded values in the Secret file can be updated using the command:
  ```bash
  echo -n 'value' | base64
