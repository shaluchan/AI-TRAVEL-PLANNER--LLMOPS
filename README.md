# 🚀LLMOps Project: Kubernetes Deployment with Docker and ELK Logging

## Project Overview

The main aim of this project is to **host a Streamlit-based application on Kubernetes using Docker on an EC2 instance**. Additionally, this project focuses on **practicing logging and monitoring services** using **ELK stack (Elasticsearch, Logstash, Kibana) and Filebeat**. This setup allows centralized log collection and visualization from all Kubernetes pods, making it easier to monitor the application and the cluster.

---

## 🔍Tech Stack

- **Docker:** Containerization of the application for easy deployment.  
- **Kubernetes (Minikube on VM):** Orchestrating containerized apps with scaling and management.  
- **EC2:** Hosting the Kubernetes cluster and Docker runtime.     
- **Filebeat:** Collects logs from pods/nodes and forwards them to Logstash.
- **Logstash:** Processes and forwards logs to Elasticsearch.  
- **Elasticsearch:** Stores and indexes logs collected from Kubernetes pods.
- **Kibana:** Provides a web interface to visualize logs stored in Elasticsearch.
---

## 📤 Project Setup

### 1. **Create VM Instance on Google Cloud / EC2**
   - Machine Type: E2, Standard, 16 GB RAM  
   - Boot Disk: Ubuntu 24.04 LTS, 256 GB  
   - Networking: Enable HTTP/HTTPS traffic 
   - Port: Open port 5601 and 8501 in inbound rules.
     
### 2. **Connect to the VM**  
   Use SSH from the browser or your terminal.

### 3. **Configure VM Instance**

- Clone my GitHub repository:

```
git clone https://github.com/shaluchan/AI-TRAVEL-PLANNER--LLMOPS.git
cd AI-TRAVEL-PLANNER--LLMOPS
```
- Install docker,minikube and kubectl by referring official docs.
- Start minikube
```
 start minikube
```
### 4. **Build and deploy the app on VM**

- Point Docker to Minikube
```
eval $(minikube docker-env)
```
- Build the image inside minikube's cluster
```
docker build -t planner-app:latest .
```
- Inject secrets directly into the cluster
```
kubectl create secret generic llmops-secrets \
  --from-literal=GROQ_API_KEY=""
```
- create deployment
```
kubectl apply -f k8s-deployment.yaml
```
- Check the pods running
```
kubectl get pods
```
- Expose Streamlit Service on EC2 Public IP via Port Forwarding
```
kubectl port-forward svc/streamlit-service 8501:80 --address 0.0.0.0
```
- Now copy external ip and :5000 and see ur app there....


## 🔄 ELK Stack Setup on Kubernetes with Filebeat 

### 1. Create a Namespace for logging
```
kubectl create namespace logging
```
### 2. Deploy Elasticsearch
```
kubectl apply -f elasticsearch.yaml    ➡️ Applies your Elasticsearch deployment configuration  
kubectl get pods -n logging            ➡️ Checks if Elasticsearch pods are up and running  
kubectl get pvc -n logging             ➡️ Checks PersistentVolumeClaims — should be in Bound state  
kubectl get pv -n logging              ➡️ Checks PersistentVolumes — should also be in Bound state  

✅ Elasticsearch setup done...
```
### 3. Deploy Kibana
```
kubectl apply -f kibana.yaml                                             ➡️ Deploys Kibana, the frontend for Elasticsearch  
kubectl get pods -n logging                                              ➡️ Wait until the Kibana pod is in Running state  
kubectl port-forward -n logging svc/kibana 5601:5601 --address 0.0.0.0   ➡️ Makes Kibana accessible at http://<your-ip>:5601  
                                                        

✅ Kibana setup done...
```
### 4. Deploy Logstash
```
kubectl apply -f logstash.yaml        ➡️ Deploys Logstash to process and forward logs  
kubectl get pods -n logging           ➡️ Ensure Logstash is running  

✅ Logstash setup done...
```
### 5. Deploy Filebeat
```
kubectl apply -f filebeat.yaml        ➡️ Deploys Filebeat to collect logs from all pods/nodes and send to Logstash  
kubectl get all -n logging            ➡️ Checks all resources (pods, services, etc.) to confirm everything is running  

✅ Filebeat setup done...
```
### 6. Setup Index Patterns in Kibana
- Open Kibana in browser → http://<your-ip>:5601
- Click "Explore on my own"
- Go to Stack Management from the left panel
- Click Index Patterns
- Create new index pattern:
- Pattern name: filebeat-*
- Timestamp field: @timestamp
- Click Create Index Pattern
- ➡️ This tells Kibana how to search and filter logs coming from Filebeat.

### 7. Explore Logs
- In the left panel, click "Analytics → Discover"
- You will see logs collected from Kubernetes cluster!
- Use filters like:
- kubernetes.container.name to filter logs from specific pods like Filebeat, Kibana, Logstash, etc.
- ✅ Done! Now you can monitor and analyze your K8s logs using ELK + Filebeat. 

