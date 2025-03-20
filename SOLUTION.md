# Two-Tier Flask App Deployment with MySQL (Without Helm)

This repository contains the necessary files and instructions to deploy a two-tier Flask application with a MySQL database using Kubernetes without Helm. The application is containerized and can be deployed on any Kubernetes cluster.

## Prerequisites

- Docker installed
- Kubernetes cluster (Minikube, Kind, or any other Kubernetes environment)
- kubectl configured to interact with your Kubernetes cluster

## Directory Structure

```
two-tier-flask-app/
├── app.py
├── Dockerfile
├── Dockerfile-multistage
├── docker-compose.yml
├── eks-manifests
├── Jenkinsfile
├── k8s
│   ├── mysql-deployment.yaml
│   ├── mysql-service.yaml
│   ├── flask-app-deployment.yaml
│   ├── flask-app-service.yaml
│   ├── mysql-pv.yaml
│   └── mysql-pvc.yaml
├── Makefile
├── message.sql
├── README.md
├── requirements.txt
└── templates
```

## Deployment Steps

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/two-tier-flask-app.git
cd two-tier-flask-app
```

### 2. Build Docker Images

Build the Docker images for the Flask application and MySQL database.

```bash
docker build -t your-dockerhub-username/flaskapp:latest -f Dockerfile .
docker build -t your-dockerhub-username/mysql:latest -f Dockerfile-multistage .
```

### 3. Push Docker Images to Docker Hub

Push the built images to Docker Hub.

```bash
docker push your-dockerhub-username/flaskapp:latest
docker push your-dockerhub-username/mysql:latest
```

### 4. Deploy MySQL Persistent Volume and Persistent Volume Claim

Create a Persistent Volume (PV) and Persistent Volume Claim (PVC) for MySQL.

```bash
kubectl apply -f k8s/mysql-pv.yaml
kubectl apply -f k8s/mysql-pvc.yaml
```

### 5. Deploy MySQL Service and Deployment

Deploy the MySQL service and deployment.

```bash
kubectl apply -f k8s/mysql-service.yaml
kubectl apply -f k8s/mysql-deployment.yaml
```

### 6. Get MySQL Cluster IP

After deploying the MySQL service, get the Cluster IP of the MySQL service:

```bash
kubectl get all
```

Copy the Cluster IP of the MySQL service (e.g., `10.96.24.184`) and update the `MYSQL_HOST` environment variable in the `flask-app-deployment.yaml` file with this IP address.

### 7. Deploy Flask App Service and Deployment

Deploy the Flask application service and deployment.

```bash
kubectl apply -f k8s/flask-app-service.yaml
kubectl apply -f k8s/flask-app-deployment.yaml
```

### 8. Verify Deployment

Check the status of the deployed pods and services.

```bash
kubectl get all
```

### 9. Access the Application

The Flask application is exposed via a NodePort service. You can access it using the IP of your Kubernetes node and the assigned port (e.g., 30004).

```bash
kubectl get svc two-tier-app-service
```

### 10. Port Forwarding (Optional)

If you want to access the application locally, you can use port forwarding.

```bash
kubectl port-forward svc/two-tier-app-service 30004:80 --address=0.0.0.0
```

Then, open your browser and navigate to `http://localhost:30004`.

## Configuration

### MySQL Configuration

The MySQL service is configured with the following environment variables:

- `MYSQL_HOST`: The IP address of the MySQL service.
- `MYSQL_USER`: The MySQL user (default: `root`).
- `MYSQL_PASSWORD`: The MySQL password (default: `admin`).
- `MYSQL_DB`: The MySQL database name (default: `mydb`).

### Flask App Configuration

The Flask application is configured to connect to the MySQL database using the environment variables mentioned above.

## Troubleshooting

- **Service Not Found**: Ensure that the service names match in your Kubernetes manifests.
- **Port Forwarding Errors**: Verify that the service port and target port are correctly specified in the service manifest.

## Cleanup

To delete the deployed resources:

```bash
kubectl delete -f k8s/flask-app-deployment.yaml
kubectl delete -f k8s/flask-app-service.yaml
kubectl delete -f k8s/mysql-deployment.yaml
kubectl delete -f k8s/mysql-service.yaml
kubectl delete -f k8s/mysql-pvc.yaml
kubectl delete -f k8s/mysql-pv.yaml
```

