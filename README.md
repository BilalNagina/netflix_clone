# Netflix Landing Page Clone — Docker + Kubernetes (Minikube)

A responsive Netflix landing page clone built with **HTML & CSS**.  
Containerized with **Docker**, deployed on **Kubernetes (Minikube)**, and exposed via **NGINX Ingress**.

---

## 📂 Project Structure
netflix-clone/
index.html
style.css
favicon.ico
Dockerfile
deployment.yaml
service.yaml
ingress.yaml
assets/

---

## ⚙️ Prerequisites
- Docker
- Minikube
- kubectl (configured for Minikube)
- Docker Hub account (for pushing images)
  
-------------------------------------------------------
## 🐳 Docker

### Dockerfile
```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
```
### Build & Run Locally
```
Build image
docker build -t netflix-clone:latest .
```
### Run container
```
docker run -d -p 8080:80 netflix-clone:latest
```
# Open http://localhost:8080

### Tag & Push to Docker Hub
```Login
docker login
```
```Tag
docker tag netflix-clone:latest bilaln/netflix-clone:v1
```
```Push
docker push bilaln/netflix-clone:v1
```
💡 Add a .dockerignore to skip unnecessary files (e.g. .git, node_modules).

-------------------------------------------------------
## ☸️ Kubernetes (Minikube)

### 1. Start cluster & enable ingress
```
   minikube start
   minikube addons enable ingress
```

### 2. Deployment (deployment.yaml)
 ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: netflix-clone-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: netflix-clone
      template:
        metadata:
          labels:
            app: netflix-clone
        spec:
          containers:
          - name: netflix-clone
            image: bilaln/netflix-clone:v1
            ports:
            - containerPort: 80
```

  ### Apply:
  ```
     kubectl apply -f deployment.yaml
     kubectl get pods -l app=netflix-clone
  ```

### 3. Service (service.yaml)
```
      apiVersion: v1
      kind: Service
      metadata:
        name: nfc-svc
      spec:
        selector:
          app: netflix-clone
        ports:
        - port: 80
          targetPort: 80
        type: LoadBalancer
```
  ### Apply:
  ```
       kubectl apply -f service.yaml
       kubectl get svc nfc-svc
  ```
   
   ### Get a quick local URL:
   ```
     minikube service nfc-svc --url
   ```
  
### 4. Ingress (ingress.yaml)
```
     apiVersion: networking.k8s.io/v1
     kind: Ingress
      metadata:
        name: nfc-ingress
        annotations:
          nginx.ingress.kubernetes.io/rewrite-target: /
      spec:
        rules:
        - host: netflix.local
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: nfc-svc
                  port:
                    number: 80
  ```
   ### Apply:
   ```
       kubectl apply -f ingress.yaml
       kubectl get ingress
   ```
### 5. Map hostname

  Find Minikube IP:
      ```
        minikube ip
      ```
  Edit /etc/hosts:
      ```
        <minikube-ip> netflix.local
      ```
  Example:
  ```
        192.168.43.2 netflix.local
  ```

  Access:
  ```
        http://netflix.local
  ```

-------------------------------------------------------   
## 🌐 Traffic Flow

### With Ingress:

  Client → DNS (netflix.local → Minikube IP)
      → Ingress Controller (port 80)
      → Service (nfc-svc)
      → Pod(s) (netflix-clone)

### Without Ingress:

  Client → NodeIP:NodePort → Service → Pod

-------------------------------------------------------
## 🧹 Cleanup
```
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
minikube stop
```





