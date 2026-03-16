A Python-based microservices application for converting **MP4 videos to MP3 audio files**. This project uses a **microservices architecture** and can be deployed on **AWS Elastic Kubernetes Service (EKS)**.

---

## Project Overview

The application consists of the following microservices:

- **Auth Service** – Handles user authentication and login.  
- **Gateway Service** – Routes API requests to the appropriate service.  
- **Converter Module** – Converts video files from MP4 to MP3.  
- **Notification Service** – Sends email notifications and supports 2-factor authentication (2FA).  
- **Database Services** – PostgreSQL for structured data and MongoDB for file metadata.  
- **Message Queue** – RabbitMQ manages conversion tasks asynchronously.

---

## Prerequisites

Before deploying the project, ensure the following tools and services are installed:

- AWS Account to create and manage EKS clusters.  
- Python for running microservices locally or building Docker images.  
- AWS CLI to manage AWS resources from the command line.  
- kubectl, the Kubernetes command-line tool.  
- Helm, a Kubernetes package manager.  
- PostgreSQL and MongoDB instances for database storage.

---

## Deployment Steps

### Database Setup

**MongoDB:** Navigate to the Helm chart folder for MongoDB, configure username and password in `values.yaml`, and run `helm install mongo .`. Connect to MongoDB using `mongosh mongodb://<username>:<password>@<nodeIP>:30005/mp3s?authSource=admin`.

**PostgreSQL:** Navigate to the Helm chart folder for PostgreSQL, configure username and password in `values.yaml`, and run `helm install postgres .`. Initialize the database with the queries in `init.sql` and connect using `psql 'postgres://<username>:<password>@<nodeIP>:30003/authdb'`.

### RabbitMQ Setup

Deploy RabbitMQ using Helm and create two queues named `mp3` and `video` to manage conversion tasks asynchronously.

### Deploy Microservices

For each microservice, navigate to its manifest folder and apply the Kubernetes configuration. Recommended deployment order is Auth Service, Gateway Service, Converter Module, and Notification Service. Use `kubectl apply -f .` in each manifest folder to deploy.

### Kubernetes Cluster Setup on AWS EKS

1. **Create IAM Roles:** Create a cluster role for EKS and a node role with the following policies: `AmazonEKSNodePolicy`, `AmazonEKS_CNI_Policy`, `AmazonEC2ContainerRegistryReadOnly`, and `AmazonEBSCSIDriverPolicy`.  
2. **Create EKS Cluster:** Configure networking including VPC and subnets, assign the IAM role, and wait for the cluster status to show `Active`.  
3. **Add Node Groups:** Select the desired instance type and number of nodes, ensure the node security group allows necessary inbound ports.  
4. **Enable EBS CSI Addon** to support persistent volumes for your cluster.

### Application Validation

After deployment, verify all pods and services are running using `kubectl get all`.

### Notification Service Configuration

Enable 2-Step Verification on your Gmail account, generate an application-specific password, and add it to `notification-service/manifest/secret.yaml` to allow email notifications.

### API Endpoints

- **Login:** `POST http://<nodeIP>:30002/login`  
- **Upload Video:** `POST http://<nodeIP>:30002/upload`  
- **Download Converted MP3:** `GET http://<nodeIP>:30002/download?fid=<file_id>`

### Cleaning Up Infrastructure

To avoid ongoing AWS costs, first delete EKS node groups, then delete the EKS cluster entirely.

---

## Notes

- RabbitMQ handles video conversion tasks asynchronously.  
- Email notifications are integrated with 2FA for security.  
- Use kubectl and Helm to manage deployments and configurations.  
- Ensure security groups allow all required ports for service communication.

---

## Quick Command Reference

- Clone the repository: `git clone https://github.com/<username>/<repo>.git` and navigate into it: `cd <repo>`.  
- Set the Kubernetes context: `aws eks update-kubeconfig --name <cluster_name> --region <aws_region>`.  
- Deploy microservices: `cd <service-folder>/manifest` and `kubectl apply -f .`.  
- Validate deployment: `kubectl get all`.  
- Clean up EKS resources: `aws eks delete-nodegroup --cluster-name <cluster_name> --nodegroup-name <nodegroup_name>` and `aws eks delete-cluster --name <cluster_name>`.
