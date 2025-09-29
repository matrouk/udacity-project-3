# Coworking Analytics – Cloud DevOps Project

## Project Overview
This project deploys a Flask-based analytics application connected to a PostgreSQL database on AWS EKS.  
The pipeline automates building, containerizing, storing, and deploying the application.

---

## Architecture
- **AWS EKS**: Hosts Kubernetes cluster
- **PostgreSQL**: Stateful database deployed as a pod + service
- **Flask App**: Analytics service exposing `/api/reports/user_visits`
- **Amazon ECR**: Stores Docker images
- **AWS CodeBuild**: CI/CD pipeline to build & push images
- **LoadBalancer Service**: Provides external access to the Flask API

---

## Prerequisites

**Tools Required:**
- **AWS CLI**: Configured with appropriate permissions
- **eksctl**: For EKS cluster management
- **kubectl**: Kubernetes command-line tool
- **Docker**: For building and running containers
- **Python 3.x**: For local development and testing
- **psql**: PostgreSQL client for database operations

---

## How to Reproduce

1. **Clone the repository:**
   ```bash
   git clone https://github.com/matrouk/udacity-project-3.git
   cd udacity-project-3
   ```

2. **Create EKS cluster:**
   ```bash
   eksctl create cluster --name my-cluster --region us-east-1 --nodegroup-name my-nodes --node-type t3.small --nodes 1 --nodes-min 1 --nodes-max 2
   ```

3. **Deploy database:**
   ```bash
   kubectl apply -f deployment/postgresql-deployment.yaml
   kubectl apply -f deployment/postgresql-service.yaml
   ```

4. **Seed database:**
   ```bash
   kubectl exec -it <postgresql-pod> -- psql -U myuser -d mydatabase
   # Run SQL scripts from db/ folder
   ```

5. **Build and push Docker image:**
   ```bash
   docker build -t coworking-analytics ./analytics
   # Tag and push to ECR (or use CodeBuild pipeline)
   ```

6. **Deploy application:**
   ```bash
   kubectl apply -f deployment/configmap.yml
   kubectl apply -f deployment/secrets.yml
   kubectl apply -f deployment/deployment.yaml
   ```

7. **Test external access:**
   ```bash
   kubectl get svc
   curl http://<EXTERNAL-IP>:5153/api/reports/user_visits
   ```

---

## Steps Completed

### 1. Cluster Setup
- Created EKS cluster using `eksctl`
- Verified nodes:
  ```bash
  kubectl get nodes -o wide
  ```

### 2. Database Deployment
- Applied `postgresql-deployment.yaml` and `postgresql-service.yaml`
- Executed SQL scripts:
  - `db/1_create_tables.sql`
  - `db/2_seed_users.sql`
  - `db/3_seed_tokens.sql`
- Verified data with `psql`

### 3. Local App + Docker
- Ran Flask app locally:
  ```bash
  python app.py
  ```
- Built Docker image:
  ```bash
  docker build -t coworking-analytics:1.0.0 ./analytics
  ```
- Verified container with environment variables and curl.

### 4. CI/CD with CodeBuild & ECR
- Created private repo in ECR: `coworking`
- Configured CodeBuild with GitHub repo and `buildspec.yaml`
- CodeBuild builds and pushes `coworking:1.0.0` and `latest` to ECR

### 5. Kubernetes Deployment
- Applied manifests:
  ```bash
  kubectl apply -f configmap.yaml
  kubectl apply -f secrets.yaml
  kubectl apply -f deployment.yaml
  ```
- Verified pods and services are running
- Exposed app via LoadBalancer → external DNS

### 6. External Test
- Accessed API externally:
  ```bash
  curl http://<ELB-DNS>:5153/api/reports/user_visits
  ```
- Received JSON output with seeded user visits ✅

---

## Verification Screenshots

Screenshots included:
- **Cluster setup** (`kubectl config current-context`, nodes)
- **PostgreSQL running** + seeded data
- **Flask app running locally**
- **Docker build** + run
- **ECR repo** + CodeBuild success
- **EKS deployment** + services
- **External API response**

---

## Rubric Mapping

This project demonstrates the following rubric requirements:

### **Container Creation (Screenshots 10-11, 14-15)**
- ✅ **Screenshot 10**: Docker build process
- ✅ **Screenshot 11**: Docker container running locally
- ✅ **Screenshot 14**: CodeBuild success (automated container build)
- ✅ **Screenshot 15**: Docker images available in ECR

### **Container Registry (Screenshots 12, 15)**
- ✅ **Screenshot 12**: ECR repository creation
- ✅ **Screenshot 15**: Container images stored in ECR registry

### **Kubernetes Cluster (Screenshots 1-2, 16-19)**
- ✅ **Screenshot 1**: Cluster context configuration
- ✅ **Screenshot 2**: EKS nodes running
- ✅ **Screenshots 16-19**: Kubernetes services, pods, and deployments

### **Database Setup (Screenshots 4-7)**
- ✅ **Screenshot 4**: Database service running
- ✅ **Screenshots 5-7**: Database tables created and seeded with data

### **Application Deployment (Screenshots 3, 16-20)**
- ✅ **Screenshot 3**: Application pods running
- ✅ **Screenshots 16-18**: Kubernetes deployment status
- ✅ **Screenshot 20**: External API access working

### **CI/CD Pipeline (Screenshots 13-14)**
- ✅ **Screenshot 13**: CodeBuild project linked to GitHub
- ✅ **Screenshot 14**: Successful automated build and deployment
- ✅ **buildspec.yaml**: Pipeline configuration file

### **Monitoring & Logging**
- ✅ **CloudWatch**: EKS cluster logs and monitoring enabled
- ✅ **Project screenshots folder**: Contains CloudWatch logs screenshots
- ✅ Application logs accessible via `kubectl logs`

---

## Notes

- Flask app is running in dev mode (for demo).
- In production, use Gunicorn / uWSGI behind a reverse proxy.

Clean up resources after testing:
```bash
eksctl delete cluster --name my-cluster
aws ecr delete-repository --repository-name coworking --force
```

---

## Author

**Hussain Almatrouk**