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
# 🚀 Deploying a Two-Tier Flask App Using Kubernetes, Kind & Helm

## 📌 Overview

This guide provides step-by-step instructions to deploy a **two-tier Flask application** using **Kubernetes (Kind) and Helm**. The deployment includes **frontend and backend services** with MySQL as the database.

---

## 🛠️ Prerequisites

Before you begin, ensure you have the following installed:

- ✅ Docker 🐳
- ✅ Kubernetes (Kind) ⛵
- ✅ Helm ⛑️
- ✅ Kubectl 📡

---

## 🔥 Step 1: Clone the Project Repository

```sh
# Clone the project from GitHub
$ git clone https://github.com/your-repo/two-tier-flask-app.git
$ cd two-tier-flask-app
```

---

## 🏗️ Step 2: Build & Push Docker Images

We will build and push **flask-app-docker-file** image to Docker Hub.

```sh
# Navigate to two-tier-flask-app folder and build the image
$ docker build -t your-dockerhub-username/flaskapp:latest .

# Push the image to Docker Hub
$ docker push your-dockerhub-username/flaskapp:latest
```

---

## 📌 Step 3 : Create Helm Charts for MySQL

```sh
$ helm create mysql-chart
$ cd mysql-chart
```

Modify `values.yaml` to set the **Docker image** and **environment variables**:

```yaml
image:
  repository: mysql
  tag: "latest"
env:
  mysqlrootpw: admin
  mysqldb: mydb
  mysqluser: admin
  mysqlpass: admin
service:
  type: ClusterIP
  port: 3306
```

Modify `deployment.yaml` to **update ports**:

```yaml
containers:
  - name:  {{ .Chart.Name }}
    image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
    env:
      - name: MYSQL_ROOT_PASSWORD
        value: {{ .Values.env.mysqlrootpw }}
      - name: MYSQL_DATABASE
        value: {{ .Values.env.mysqldb }}
      - name: MYSQL_USER
        value: {{ .Values.env.mysqluser }}
      - name: MYSQL_PASSWORD
        value: {{ .Values.env.mysqlpass }}
```

### 📌 Generate Flask-App Helm Chart

```sh
$ helm create flask-app-chart
$ cd flask-app-chart
```

Modify `values.yaml` to set the **Docker image**:

```yaml
image:
  repository: your_docker_hub_username/flaskapp
  pullPolicy: IfNotPresent
  tag: "latest"
env:
  mysqlhost: 10.96.24.184  # it is a Cluster IP of MySQL by running command [kubectl get svc -n mysql]
  mysqldb: mydb
  mysqluser: admin
  mysqlpass: admin
service:
  type: NodePort  
  port: 80
  targetPort: 5000
  nodePort: 30008  
```

Modify `deployment.yaml` to **update ports**:

```yaml
containers:
  - name:  {{ .Chart.Name }}
    image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
    env:
      - name: MYSQL_HOST
        value: {{ .Values.env.mysqlhost }}
      - name: MYSQL_DATABASE
        value: {{ .Values.env.mysqldb }}
      - name: MYSQL_USER
        value: {{ .Values.env.mysqluser }}
      - name: MYSQL_PASSWORD
        value: {{ .Values.env.mysqlpass }}
    ports:
      - name: http
        containerPort: {{ .Values.service.targetPort }}
```

Modify `service.yaml` to configure the service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
spec:
  selector:
    app: flask-app
  type: {{ .Values.service.type }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      nodePort: {{ .Values.service.nodePort }}  # Only used if type is NodePort
```

---

## 📦 Step 5: Package & Install Helm Charts

### 🔹 Package the MySQL Chart

```sh
$ cd two-tier-flask-app
$ helm package mysql-chart
$ helm install mysql-chart ./mysql-chart
```

📌 **Expected Output:**

```
mysql-chart deployed successfully! 🎉
```

### 🔹 Package the Flask-App Chart

```sh
$ cd two-tier-flask-app
$ helm package flask-app-chart
$ helm install flask-app-chart ./flask-app-chart
```

📌 **Expected Output:**

```
flask-app-chart deployed successfully! 🚀
```

---

## 🔄 Step 6: Verify Deployments

Check all running **pods**:

```sh
$ kubectl get pods
```

📌 **Expected Output:**

```
NAME                             READY   STATUS    RESTARTS   AGE
flask-app-chart-xxxxxxx          1/1     Running   0          30s
flask-app-chart-xxxxxxx          1/1     Running   0          30s
mysql-xxxxxxx                    1/1     Running   0          30s
```

Check **services**:

```sh
$ kubectl get svc
```

📌 **Expected Output:**

```
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)

flask-app-service     ClusterIP   10.96.0.2        <none>        3000/TCP
mysql-service        ClusterIP   10.96.0.3        <none>        3306/TCP
```

---

## 🌎 Step 7: Access the Application

If using **Ingress**:

```sh
$ echo "http://your-ingress-domain"
```

If using **NodePort**:

```sh
$ kubectl get svc flask-app-service -o=jsonpath='{.spec.ports[0].nodePort}'
$ echo "http://localhost:<NodePort>"
```

📌 **Expected Output:**

```
http://localhost:3000
```

---

## 🎯 Conclusion

Congratulations! 🎉 You have successfully deployed a **two-tier Flask app** using **Kubernetes (Kind) and Helm**. You can now access your app via the exposed URL.

Happy coding! 🚀🔥
