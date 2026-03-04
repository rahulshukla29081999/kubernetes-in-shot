# 1️⃣ Environment Setup
docker --version
docker ps
kubectl version --client
kind version

# Add Docker Permission (Non-root User)
sudo usermod -aG docker $USER
newgrp docker
bash
# 2️⃣ Create Kind Cluster
mkdir kind_cluster
cd kind_cluster

# Create config.yml
vim config.yml

# Create Cluster
kind create cluster --name tws-cluster --config=config.yml

# Verify Cluster
kubectl cluster-info --context kind-tws-cluster
kubectl get nodes
bash
# 3️⃣ Basic Kubernetes Commands
kubectl get namespaces
kubectl get pods
kubectl get pods -n kube-system
kubectl get nodes
bash
# 4️⃣ Nginx Namespace & Pod
# CLI Way
kubectl create ns nginx

# YAML Way
vim namespace.yml
kubectl apply -f namespace.yml

# Create Pod
vim pod.yml
kubectl apply -f pod.yml
kubectl get pods -n nginx

# Describe Pod
kubectl describe pod nginx-pod -n nginx

# Exec Into Pod
kubectl exec -it nginx-pod -n nginx -- bash
bash
# 5️⃣ Deployment & Scaling
vim deployment.yml
kubectl apply -f deployment.yml
kubectl get deployment -n nginx

# Scale Deployment
kubectl scale deployment/nginx-deployment -n nginx --replicas=5
kubectl scale deployment/nginx-deployment -n nginx --replicas=1
bash
# 6️⃣ Update Image (Rolling Update)
kubectl set image deployment/nginx-deployment -n nginx nginx=nginx:1.26.2
kubectl set image deployment/nginx-deployment -n nginx nginx=nginx:latest
bash
# 7️⃣ ReplicaSet
vim replicasets.yml
kubectl apply -f replicasets.yml
kubectl get replicasets -n nginx
bash
# 8️⃣ DaemonSet
vim daemonsets.yml
kubectl apply -f daemonsets.yml
kubectl get pods -n nginx -o wide
bash
# 9️⃣ Job
vim job.yml
kubectl apply -f job.yml
kubectl get jobs -n nginx
kubectl logs -n nginx <job-pod-name>
bash
# 🔟 CronJob
vim cron-job.yml
kubectl apply -f cron-job.yml
kubectl get cronjob -n nginx
bash
# 1️⃣1️⃣ Services & Port Forward
vim service.yml
kubectl apply -f service.yml
kubectl get svc -n nginx

# Port Forward
kubectl port-forward service/nginx-service -n nginx 8081:80 --address=0.0.0.0
bash
# 1️⃣2️⃣ Ingress Setup (Kind)
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
kubectl get svc -n ingress-nginx

# Apply Ingress
vim ingress.yml
kubectl apply -f ingress.yml

# Port Forward Ingress
kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 8080:80 --address=0.0.0.0
bash
# 1️⃣3️⃣ Persistent Volume & PVC
vim persistentVolume.yml
kubectl apply -f persistentVolume.yml
kubectl get pv

vim persistentVolumeClaim.yml
kubectl apply -f persistentVolumeClaim.yml
kubectl get pvc
bash
# 1️⃣4️⃣ Metrics Server (kubectl top)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl -n kube-system edit deployment metrics-server
kubectl -n kube-system rollout restart deployment metrics-server

# Check Metrics
kubectl top node
kubectl top pod
bash
# 1️⃣5️⃣ Node Taints & Tolerations
kubectl taint node tws-cluster-worker prod=true:NoSchedule

# Remove Taint
kubectl taint node tws-cluster-worker prod=true:NoSchedule-
bash
# 1️⃣6️⃣ Apache Deployment + HPA
mkdir apache
cd apache

kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml

# Scale
kubectl scale deployment apache-deployment -n apache --replicas=3

# Horizontal Pod Autoscaler
vim hpa.yml
kubectl apply -f hpa.yml
kubectl get hpa -n apache

# Load Generator
kubectl run -it load-generator --image=busybox -n apache -- /bin/sh
bash
# 1️⃣7️⃣ Vertical Pod Autoscaler (VPA)
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh

# Apply VPA
kubectl apply -f vpa.yml
kubectl get vpa -n apache
bash
# 1️⃣8️⃣ MySQL StatefulSet
mkdir mysql
cd mysql

kubectl apply -f namespace.yml
kubectl apply -f service.yml
kubectl apply -f statefulset.yml
kubectl get pods -n mysql

# Exec into MySQL Pod
kubectl exec -it mysql-statefulset-0 -n mysql -- bash
bash
# 1️⃣9️⃣ Docker + Django Notes App Deployment
git clone https://github.com/LondheShubham153/django-notes-app.git
cd django-notes-app
git checkout dev

# Build Docker Image
docker build -t notes-app-k8s .

# Tag & Push Image
docker login
docker tag notes-app-k8s:latest <your-dockerhub-username>/notes-app-k8s:latest
docker push <your-dockerhub-username>/notes-app-k8s:latest

# Deploy in Kubernetes
kubectl apply -f namespace.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml

# Port Forward
kubectl port-forward service/notes-app-service -n notes-app 80
