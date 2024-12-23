# User Management Application Deployment on Kubernetes

This repository contains the Kubernetes manifests to deploy a very simple user management application consisting of a MySQL database and two application components: Admin and User portals. Below is a detailed overview of the files and their functionality.

---

## Project Structure

### 0. Namespaces
- **File**: `00-namespace.yaml`
- **Purpose**: Defines the namespaces for resources.

### 1. Secrets
- **File**: `06-UserMgmt-Secret.yaml`
- **Purpose**: Defines the secrets required for the database connection and application security keys. Secrets are stored as base64 encoded values.

### 2. Storage Class
- **File**: `01-storage-class.yaml`
- **Purpose**: Configures an NFS storage class for dynamic provisioning of Persistent Volumes.

### 3. Persistent Volume Claim (PVC)
- **File**: `02-persistent-volume-claim.yaml`
- **Purpose**: Claims a persistent volume using the defined NFS storage class. Ensures shared storage for the MySQL database.

### 4. ConfigMap
- **File**: `03-UserMgmt-ConfigMap.yaml`
- **Purpose**: Provides an SQL script to initialize the MySQL database with the required schema and a default admin user.

### 5. MySQL Deployment
- **File**: `04-mysql-deployment.yaml`
- **Purpose**: Deploys a MySQL server with the necessary configuration for database persistence and initialization using the ConfigMap.

### 6. MySQL ClusterIP Service
- **File**: `05-mysql-clusterip-service.yaml`
- **Purpose**: Creates a ClusterIP service for internal communication with the MySQL database.

### 7. Admin Component Deployment and Service
- **Files**: 
  - `07-UserMgmtAdmin-Deployment.yaml`
  - `09-UserMgmtAdmin-Service.yaml`
- **Purpose**:
  - Deployment: Defines the Admin component with environment variables from the Secret, readiness, and liveness probes.
  - Service: Exposes the Admin component via NodePort.

### 8. User Component Deployment and Service
- **Files**: 
  - `08-UserMgmtUser-Deployment.yaml`
  - `10-UserMgmtUser-Service.yaml`
- **Purpose**:
  - Deployment: Defines the User component with similar configurations to the Admin component.
  - Service: Exposes the User component via NodePort.

### 9. Network policy for db
- **Files**: 
  - `11-networkpolicy-for-db.yaml`
- **Purpose**:
  -  Restrict MySQL access, allowing only connections from the `usermgm` namespace to the database in the `database` namespace.

---

## Deployment Steps

0. **Create the namespace resources**:
   ```bash
   kubectl apply -f 00-namespace.yaml
   ```

1. **Create the Storage Class**:
   ```bash
   kubectl apply -f 01-storage-class.yaml
   ```
2. **Create the Persistent Volume Claim**:
   ```bash
   kubectl apply -f 02-persistent-volume-claim.yaml
   ```
3. **Create the ConfigMap**:
   ```bash
   kubectl apply -f 03-UserMgmt-ConfigMap.yaml
   ```
4. **Deploy the MySQL Server**:
   ```bash
   kubectl apply -f 04-mysql-deployment.yaml
   ```
5. **Expose MySQL Server**:
   ```bash
   kubectl apply -f 05-mysql-clusterip-service.yaml
   ```
6. **Create the Secrets**:
   ```bash
   kubectl apply -f 06-UserMgmt-Secret.yaml
   ```
7. **Deploy and Expose the Admin Component**:
   ```bash
   kubectl apply -f 07-UserMgmtAdmin-Deployment.yaml
   kubectl apply -f 09-UserMgmtAdmin-Service.yaml
   ```
8. **Deploy and Expose the User Component**:
   ```bash
   kubectl apply -f 08-UserMgmtUser-Deployment.yaml
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

## Notes

- Ensure that the NFS server specified in the `01-storage-class.yaml` file is reachable.
- The base64 encoded values in the Secret file can be updated using the command:
  ```bash
  echo -n 'value' | base64
