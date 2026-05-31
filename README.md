# 💬 ChatPulse - Premium Real-Time Full-Stack Chat Application

[![Kubernetes Supported](https://img.shields.io/badge/Kubernetes-v1.28+-blue?logo=kubernetes&logoColor=white&style=flat-square)](https://kubernetes.io/)
[![Docker Verified](https://img.shields.io/badge/Docker-Verified-blue?logo=docker&logoColor=white&style=flat-square)](https://www.docker.com/)
[![React Enabled](https://img.shields.io/badge/React-v18.3-blue?logo=react&logoColor=white&style=flat-square)](https://react.dev/)
[![NodeJS Powered](https://img.shields.io/badge/Node.js-v18-green?logo=nodedotjs&logoColor=white&style=flat-square)](https://nodejs.org/)
[![License MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

**ChatPulse** is a state-of-the-art, secure, and production-ready real-time chat application built on the MERN stack. Designed with scalability and reliability in mind, this project is fully containerized with Docker and features a robust Kubernetes orchestration system for high-availability deployments.

---

## 🚀 Key Features

*   **Real-time Messaging**: Instant message delivery and typing indicators powered by **Socket.io**.
*   **User Authentication**: Secure JWT-based auth stored in HTTP-only, SameSite-strict cookies.
*   **Secure by Design**: Configurable cookie security, dynamic CORS rules, and sensitive data stored in Kubernetes Secrets.
*   **Persistent Storage**: MongoDB database coupled with Kubernetes PersistentVolumes for data safety.
*   **Cloud Integrations**: Fully integrated with **Cloudinary** for scalable image hosting and profile customization.
*   **Production Orchestration**: Pre-configured Kubernetes resources (Pods, Services, PV/PVC, Ingress) with liveness & readiness health checks, and database initialization barriers.

---

## 🏗️ System Architecture

```mermaid
graph TD
    User([Browser Client]) -->|HTTP/WS| Ingress[Kubernetes Nginx Ingress]
    
    subgraph K8s [Kubernetes Namespace: chat-app]
        Ingress -->|/| FrontendService[Service: frontend]
        Ingress -->|/api| BackendService[Service: backend]
        Ingress -->|/socket.io| BackendService
        
        FrontendService -->|Port 80| FrontendPod[Deployment: frontend-pod]
        BackendService -->|Port 5001| BackendPod[Deployment: backend-pod]
        
        BackendPod -->|Init Wait| MongoService[Service: mongodb]
        BackendPod -->|Mongoose Connection| MongoService
        
        MongoService -->|Port 27017| MongoPod[Deployment: mongodb-pod]
        MongoPod -->|Data Volume| MongoPVC[Claim: mongodb-pvc]
        MongoPVC -->|Storage Bind| MongoPV[PV: mongodb-pv]
    end
    
    BackendPod -.->|Upload Images| Cloudinary[Cloudinary API]
    MongoPV -.->|Write Files| HostDir[(Host Node Path: /mnt/data/mongodb)]
```

---

## 🛠️ Tech Stack

*   **Frontend**: React (Vite), TailwindCSS, DaisyUI, Zustand (State Management), Lucide React.
*   **Backend**: Node.js, Express, MongoDB (Mongoose), Socket.io, Cloudinary SDK.
*   **Proxy & Serve**: Nginx (configured for client-side routing, static file compression, and websocket proxying).
*   **DevOps & Deployment**: Docker, Docker Compose, Kubernetes (Ingress-Nginx, hostPath PVs, persistent volume claims).

---

## 🎛️ Environment Variables

### Local Dev (`.env` in Root)
Create a `.env` file in the root folder for local Docker Compose development:

```env
PORT=5001
NODE_ENV=production
MONGODB_URI=mongodb://root:admin@mongo:27017/chatApp?authSource=admin&retryWrites=true&w=majority
JWT_SECRET=your_super_secret_jwt_key
CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
CLOUDINARY_API_KEY=your_cloudinary_api_key
CLOUDINARY_API_SECRET=your_cloudinary_api_secret
```

### Kubernetes Secret Configuration (`k8s/secrets.yml`)
Kubernetes uses Base64 encoded values. Encode your secrets using WSL/Git Bash:
```bash
echo -n "your_secret_value" | base64
```
Then populate the decoded placeholders in [secrets.yml](file:///wsl.localhost/Ubuntu/home/abdullah/Kubernetes-in-one-shot/projects/full-stack_chatApp/k8s/secrets.yml).

---

## 📦 Getting Started

### Option 1: Run Locally with Docker Compose

1. Clone your repository:
   ```bash
   git clone https://github.com/AbdullahbinAmin/full-stack_chatApp.git
   cd full-stack_chatApp
   ```
2. Start the stack:
   ```bash
   docker-compose up -d --build
   ```
3. Access the web app at `http://localhost:8080`.

---

### Option 2: Deploy on Kubernetes (WSL/Kind/Minikube)

We have created premium K8s manifests under `/k8s`. Make sure your cluster has an **Ingress Controller** enabled (e.g. `minikube addons enable ingress` or `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/...`).

1. **Build & Push Your Docker Images**:
   Build the images with your Docker Hub registry tag and push them:
   ```bash
   # Build & push backend
   cd backend
   docker build -t abdullahbinamin/chatapp-backend:latest .
   docker push abdullahbinamin/chatapp-backend:latest
   
   # Build & push frontend
   cd ../frontend
   docker build -t abdullahbinamin/chatapp-frontend:latest .
   docker push abdullahbinamin/chatapp-frontend:latest
   ```

2. **Deploy the Manifests**:
   Run the following commands inside WSL to spin up the services:
   ```bash
   # Create namespace
   kubectl apply -f k8s/namespace.yml
   
   # Create storage configuration (PV & PVC)
   kubectl apply -f k8s/mongodb-pv.yml
   kubectl apply -f k8s/mongodb-pvc.yml
   
   # Deploy secrets
   kubectl apply -f k8s/secrets.yml
   
   # Start database
   kubectl apply -f k8s/mongodb-service.yml
   kubectl apply -f k8s/mongodb-deployment.yml
   
   # Start backend & frontend services
   kubectl apply -f k8s/backend-service.yml
   kubectl apply -f k8s/backend-deployment.yml
   kubectl apply -f k8s/frontend-service.yml
   kubectl apply -f k8s/frontend-deployment.yml
   
   # Deploy Ingress controller configuration
   kubectl apply -f k8s/ingress.yml
   ```

3. **Domain Configuration**:
   Add the ingress host to your local machine's `hosts` file (`C:\Windows\System32\drivers\etc\hosts` or `/etc/hosts` in WSL):
   ```text
   127.0.0.1  abdullahbinamin.qzz.io
   ```
   Now visit `http://abdullahbinamin.qzz.io` in your browser!

---

## 🔧 Git Fork Disconnection Guide

To disconnect this repository from the original repository (remove the "forked from" relationship on GitHub) and make it your own standalone repository:

1. **Delete and Recreate on GitHub**:
   - Go to GitHub and delete your current forked repository `full-stack_chatApp`.
   - Create a new, blank repository on your account named `full-stack_chatApp` (do **not** check "Initialize this repository with a README").

2. **Update Remotes & Push**:
   Open WSL and run the following git commands:
   ```bash
   # Rename the old origin (or delete it)
   git remote remove origin
   
   # Add your new, independent repository as origin
   git remote add origin https://github.com/AbdullahbinAmin/full-stack_chatApp.git
   
   # Push files and set upstream
   git push -u origin main
   ```
   Your repository is now completely independent with its own history!

---

## 📚 Project Snapshots

<table border="0">
  <tr>
    <td><b align="center">Real-Time Chat Dashboard</b><br><img src="frontend/public/chat.png" width="400" alt="Chat UI"></td>
    <td><b align="center">Profile Settings & Cloudinary Uploads</b><br><img src="frontend/public/settings.png" width="400" alt="Settings UI"></td>
  </tr>
  <tr>
    <td><b align="center">Secure User Login</b><br><img src="frontend/public/login.png" width="400" alt="Login UI"></td>
    <td><b align="center">Graceful Session Logout</b><br><img src="frontend/public/logout.png" width="400" alt="Logout UI"></td>
  </tr>
</table>

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.
