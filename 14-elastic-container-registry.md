# Elastic Container Registry (ECR)

In this section, you will learn how Amazon ECR helps in managing Docker Images.

- Overview of ECR
- Build the Docker image and push it to Amazon ECR
- Understand how to use the ECR hosted images (repo) in app manifests
- Deploy the application on EKS cluster

## 01. Amazon ECR Overview

- Amazon ECR is a fully managed docker registry that makes it easy for developers, devops engg to store, manage and deploy docker container images.
- Pricing: No upfront price or commitment. You pay only for the amount of storage you consume.

- **Key Terminologies**
  - Registry
  - Repository (Image)
  - Repository Policies
  - Authorization Token

## 02. Setup machine to communicate with Amazon ECR

- Install and Configure AWS CLI (v2)
- Install Docker Client

## 03. Create a new Amazon ECR Repository

### 3.1 Create an ECR Repo using AWS Console (GUI)

- AWS Management Console >> **Elastic Container Registry** >> From left-side panel, under **Private registry** section, select **Repositories** >> Click **Create Repository**
  - Repository name: aws-ecr-nginx
  - Image tag mutability: Mutable
  - Encryption configuration: AES-256

### 3.2 (Optional) Create and ECR Repo using AWS CLI

```
# Syntax - To create an ECR Repository
aws ecr create-repository --repository-name <your-repo-name> --region <your-region>

# Example - To create an ECR Repository
aws ecr create-repository --repository-name aws-ecr-nginx --region us-east-1
```

## 04. Build Docker Image and Push it to Amazon ECR

### 4.1 Build Docker Image

- Create a new file _index.html_ to be packaged inside docker image:

```
<!DOCTYPE html>
<html>
<head>
  <title>Display IP Address</title>
  <style>
    body {
      background-color: #FFCC00;
    }

    h1 {
      font-family: "Comic Sans MS";
      text-align: center;
      padding-top: 140px;
      font-size: 60px;
      margin: -15px;
    }

    p {
      font-family: sans-serif;
	  font-size: 48px;
      color: #907400;
      text-align: center;
    }
  </style>
</head>

<body>
  <h1 id=ipText>Containers are Awesome..!</h1>
  <p>This is a Containerize App - By Bindesh</p>
</body>

</html>
```

- Save the above file as _index.html_.

- Create a **Dockerfile** to package the above app:

```
FROM nginx
COPY index.html /usr/share/nginx/html
```

- Now, let's build the image:

```
# Build the docker image
docker build -t <ECR-REPOSITORY-URI>:<TAG> .
docker build -t 154511248559.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0 .

# List all the Docker Images
docker image ls

# Run Docker Image locally & Test
docker run --name <name-of-container> -p 80:80 --rm -d <ECR-REPOSITORY-URI>:<TAG>
docker run --name nginx-container -p 80:80 --rm -d 154511248559.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx:1.0.0


# List all the running containers
docker container ls
OR
docker ps

# Access Application over port 80 of your docker host | open ingress traffic on port 80 of SG
http://<docker_host_public_ip>:80

# Stop the docker container
docker stop nginx-container
```

## 05. Push the Docker Image to Amazon ECR

### 5.1 Login to ECR using AWS CLI

```
# Syntax - Get the login password of Amazon ECR repo
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <ECR-REPOSITORY-URI>

# Example - Get the login password of Amazon ECR repo
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 154511248559.dkr.ecr.us-east-1.amazonaws.com/aws-ecr-nginx
```

### 5.2 Push the Docker Image to Amazon ECR

```
# Syntax - To push the docker image to ECR
docker push <ECR-REPOSITORY-URI>:<TAG>

# Example - Push the above create docker image to ECR
docker push 154511248559-east-1.amazonaws.com/aws-ecr-nginx:1.0.0
```

- Now, verify the newly pushed docker image on AWS ECR from AWS Management console.
- Also, verify the vulnerability scanned results.

## 06. Consuming ECR hosted Images in Amazon EKS

### 6.1 Develop and Review K8s manifests to be deployed

### 6.2 Deploy the manifests

### 6.3 Verify the Application access over the web
