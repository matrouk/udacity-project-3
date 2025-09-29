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
- Cluster setup (`kubectl config current-context`, nodes)
- PostgreSQL running + seeded data
- Flask app running locally
- Docker build + run
- ECR repo + CodeBuild success
- EKS deployment + services
- External API response

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