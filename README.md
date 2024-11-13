# Odoo Deployment on Minikube

This repository provides a simple setup to deploy Odoo community (free) and PostgreSQL on Minikube using Kubernetes manifests. Follow the steps below to set up a basic Odoo deployment with PostgreSQL as the database backend, and expose Odoo via Minikube.

## Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/) installed and running.
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed to manage the Kubernetes cluster.

## Files

- `odoo-deployment.yaml`: Defines the deployment, service, and configurations for Odoo.
- `postgres-deployment.yaml`: Defines the deployment, service, and configurations for PostgreSQL.

---

## Step 1: Deploy PostgreSQL

### 1.1 Configure Database Credentials

In `postgres-deployment.yaml`, configure environment variables to set the PostgreSQL user, password, and database name that Odoo will use:

```yaml
env:
  - name: POSTGRES_USER
    value: "odoo"
  - name: POSTGRES_PASSWORD
    value: "odoo"
  - name: POSTGRES_DB
    value: "postgres"
```
These credentials allow Odoo to connect to the PostgreSQL database. Ensure the username, password, and database name match the values in the Odoo deployment configuration.

### 1.2 Deploy PostgreSQL

Run the following command to apply the PostgreSQL configuration:
```bash
kubectl apply -f postgres-deployment.yaml
```

Confirm the deployment:
```bash
kubectl get pods
```
This should show a running PostgreSQL pod.

---

## Step 2: Deploy Odoo

### 2.1 Configure Odoo to Connect to PostgreSQL

In =odoo-deployment.yaml=, define the environment variables to set up the database host, username, and password:
```yaml
env:
  - name: HOST
    value: "db"  # Service name for PostgreSQL
  - name: USER
    value: "odoo"
  - name: PASSWORD
    value: "odoo"
```
Ensure the values here match those configured in the PostgreSQL deployment.

### 2.2 Expose Odoo

To access Odoo from outside the cluster, expose the Odoo service as a =NodePort=:

```yaml
spec:
  type: NodePort
  ports:
    - port: 8069
      targetPort: 8069
      nodePort: 30007  # Port that will be exposed
```

### 2.3 Deploy Odoo

Run the following command to apply the Odoo configuration:

```bash
kubectl apply -f odoo-deployment.yaml
```
Confirm the deployment:
```
kubectl get pods
```

This should show a running Odoo pod.

---

## Step 3: Access Odoo on Minikube

To access Odoo, use Minikube's IP and the NodePort configured in the =odoo-deployment.yaml= file.

## 3.1 Get Minikube IP

Run the following command to get the Minikube IP:

```yaml
minikube ip
```

### 3.2 Access Odoo
Use the Minikube IP and NodePort in your browser to access Odoo:

```plaintext
http://<minikube-ip>:30007
```

This should open the Odoo interface.

---

## Example YAML Files

Here’s a breakdown of the example files:

### postgres-deployment.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  ports:
    - port: 5432
  selector:
    app: postgres
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:13
          env:
            - name: POSTGRES_USER
              value: "odoo"
            - name: POSTGRES_PASSWORD
              value: "odoo"
            - name: POSTGRES_DB
              value: "postgres"
          ports:
            - containerPort: 5432
```

### odoo-deployment.yaml

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: odoo
spec:
  type: NodePort
  ports:
    - port: 8069
      targetPort: 8069
      nodePort: 30007
  selector:
    app: odoo
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: odoo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: odoo
  template:
    metadata:
      labels:
        app: odoo
    spec:
      containers:
        - name: odoo
          image: odoo:16.0
          env:
            - name: HOST
              value: "db"
            - name: USER
              value: "odoo"
            - name: PASSWORD
              value: "odoo"
          ports:
            - containerPort: 8069
```
---

## Additional Notes

- **Persistence**: To make data persistent, configure persistent volumes.
- **Customization**: Modify configurations (e.g., =POSTGRES_DB=) as per requirements.
- **Scaling**: Update =replicas= in the deployment specs to increase redundancy.

With these steps and configuration files, you can deploy and access Odoo on Minikube, connected to PostgreSQL as a backend service.
