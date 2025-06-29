# MERN Stack Kubernetes Deployment with Helm and Jenkins

This repository contains a complete Kubernetes deployment setup for a MERN (MongoDB, Express.js, React, Node.js) application using:
- **Kubernetes** for container orchestration
- **Helm** for templated, reusable Kubernetes manifests
- **Docker** for containerizing the applications
- **Jenkins** (optional) for CI/CD automation
---

## Directory Structure
kube-assignments/<br>
├── backend_kube/         # Kubernetes manifests for the backend<br>
│   └── ...               # (From https://github.com/tanujbhatia24/backend_kube.git)<br>
├── frontend_kube/        # Kubernetes manifests for the frontend<br>
│   └── ...               # (From https://github.com/tanujbhatia24/frontend_kube.git)<br>
├── mern-chart/           # Helm chart for the MERN stack<br>
│   ├── templates/<br>
│   │   ├── backend.yaml<br>
│   │   ├── frontend.yaml<br>
│   │   └── mongo.yaml<br>
│   ├── Chart.yaml<br>
│   └── values.yaml<br>
├── jenkinsfile           # CI/CD pipeline using Jenkins<br>
└── README.md             # You're here!<br>

- You need to clone backend & frontend repos inside project directory if you want it to work locally.
---

## Docker Image Build & Push

### Frontend

```bash
go to frontend_kube dir
docker build -t tanujbhatia24/frontend:latest .
docker push tanujbhatia24/frontend:latest
```
### Backend
```bash
go to backend_kube dir
docker build -t tanujbhatia24/backend:latest .
docker push tanujbhatia24/backend:latest
```
### Deploy with Helm
```bash
got mern-chart dir
helm upgrade --install mern-app . --namespace mern --create-namespace
```
### Verify Deployment
```bash
kubectl get all -n mern
```
- Make sure all pods are 1/1 READY and services are running (frontend-service, backend-service, mongo).  
---

## Access the App 
### Test via Port Forwarding
```bash
kubectl port-forward svc/frontend-service 8080:80 -n mern
kubectl port-forward svc/backend-service 3000:3000 -n mern
kubectl port-forward svc/mongo 28017:27017 -n mern
```
Frontend App available at: http://localhost:8080<br>
Backend App available at: http://localhost:3000<br>
Mongo DB available at: mongodb://localhost:28017<br>

### Test via Minikube (OPTIONAL)
```bash
minikube service frontend-service -n mern
```
- This will open your browser or give you a URL — check the site.
---

## Configuration
### .env file in Backend
```bash
ATLAS_URI=mongodb://mongo:27017/blog_mern
```
### .env file in Frontend
```bash
REACT_APP_API_BASE_URL=http://backend-service:3000
```
---

## Jenkins configuration
### Pipeline setup
1. Create Jenkinsfile inside your project directory.
2. Create dockerhub credentials in jenkins.
3. Create jenkins pipeline.
### Validation commands
Compare deploy version of helm
```bash
helm history chart_name -n namespace_name
```
EXAMPLE 
```bash
helm history mern-chart -n mern
```
---


## Validation Snapshots
- Docker Images validation snanshot<br>
![image](https://github.com/user-attachments/assets/a5ea3f75-64ba-49b7-be20-48ebbc275298)<br>

- Kubectl pod & service validation snapshot<br>
![image](https://github.com/user-attachments/assets/4d91cb13-ec33-4df7-80e1-41d0c125f175)<br>

- Frontend url validation snapshot<br>
Port forwarding<br>
![image](https://github.com/user-attachments/assets/71d26b2d-2afd-4ef5-be15-eb349cfc1c64)<br>
![image](https://github.com/user-attachments/assets/67a9f6cd-e594-4727-88df-25442e67022d)<br>
Minikube<br>
![image](https://github.com/user-attachments/assets/787ff36c-54e7-4fc1-bb66-1ff6777c9d3c)<br>
![image](https://github.com/user-attachments/assets/da1b5a65-2117-471b-a1c9-e8202223ac93)<br>

- Backend url validation snapshot<br>
![image](https://github.com/user-attachments/assets/88d3e837-992a-49b5-97bf-be0ec78544c1)<br>
![image](https://github.com/user-attachments/assets/96a1fd77-96c6-4dd6-b02f-ecb0ae58af84)<br>

- Mongo DB validation snapshot<br>
<img width="418" alt="image" src="https://github.com/user-attachments/assets/f4b1bc55-0cb2-4166-8f7f-ac4440e661c3" /><br>
<img width="528" alt="image" src="https://github.com/user-attachments/assets/008f2368-dcdc-4bee-8f93-0bb32395299a" /><br>

- Pods count before jenkins pipeline trigger snapshot<br>
<img width="323" alt="image" src="https://github.com/user-attachments/assets/8266dc7a-35ab-4525-be13-a0acf5c2c8cc" /><br>

- Pods Count after jenkins pipeline trigger snapshot (replicas set to 2)<br>
![image](https://github.com/user-attachments/assets/99d362d4-6fa4-4953-9b9d-339c2b65bbb8)<br>

- Jenkins validation snapshot<br>
![image](https://github.com/user-attachments/assets/d4078041-b93b-4f12-abec-5b3d52a6ef10)<br>
![image](https://github.com/user-attachments/assets/ee4fb4c7-a27b-40b7-abd9-972edd0923c2)<br>
---

## Author<br>
Tanuj Bhatia<br>
DockerHub: tanujbhatia24
