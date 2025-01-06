# 🚀 CI/CD Pipeline for AWS EKS, ECR, Docker, Jenkins, and AWS CLI

## 📝 Project Summary

This project demonstrates an **automated CI/CD pipeline** built using **Jenkins**, **AWS ECR**, **AWS EKS**, **Docker**, **AWS EC2**, **Load Balancer**, and **AWS CLI**. This pipeline ensures that every time developers push code to the main branch, a new Docker image is built, tested, stored, and deployed seamlessly to AWS EKS with **zero downtime** and **high availability**.

## 🛠️ Tools Used

- **Jenkins**: Automates the CI/CD pipeline.
- **Docker**: Containerizes the application.
- **AWS ECR**: Stores Docker images securely.
- **AWS EKS**: Manages Kubernetes for deployment and scaling.
- **AWS CLI**: Interacts with AWS services from the command line.
- **Kubernetes**: Manages container orchestration and application deployment.
- **AWS EC2**: As a jenkins server.
- **AWS Load Balancer**: Distributes traffic accros the pods efficiently.

---

## 🏗️ How the Pipeline Works
![Jenkins Pipeline View](stage_output.png) 

1. **Checkout Stage**: The Jenkins polls the repository for changes and checks out the latest code from the main branch. **main branch**.
2. **Code Testing**: Installs dependencies and runs tests using **Mocha** to ensure everything works smoothly.
3. **Docker Build**: Builds a **Docker image** from the application code and tags it with the Jenkins build number.
4. **Push to AWS ECR**: The Docker image is pushed to **AWS ECR**, where it’s securely stored.
5. **Deploy to AWS EKS**: The Docker image is deployed to **AWS EKS**. If the deployment exists, it updates the image; if not, it creates a new one.
6. **Rolling Update in Kubernetes**: Kubernetes performs a **rolling update**, ensuring that the application remains available without downtime.
8. **Rollback**: If any issue arises, Kubernetes allows you to easily **rollback** to a previous version of the application.

---

## 🔄 Features of the Pipeline

- **Zero Downtime Deployment**: The application is updated seamlessly without any downtime, thanks to Kubernetes **rolling updates**.
- **Rollback**: In case of an issue, the application can be **rolled back** to a previous stable version using Kubernetes.
- **Fully Automated**: The entire process from code push to deployment is automated, reducing manual intervention.


---

## Jenkins Pipeline Configuration 📋

### Key Features:

- Automatically builds and pushes Docker images with a unique tag (`BUILD_NUMBER`).
- Deploys to AWS EKS using Kubernetes manifests.
- Ensures zero-downtime updates with Kubernetes rollouts.

### Jenkinsfile

```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME = "public.ecr.aws/x9f9n9e7/aws_cicd_pipeline_ecr"
        IMAGE_TAG = "node_app_DevOps_Proj_v0.${env.BUILD_NUMBER}"
        K8S_DEPLOYMENT = "node-app-aws-cicd-deployment"
        AWS_REGION = "ap-south-1"
        cred = credentials('aws-cred')
    }
    stages {
        stage('Checkout Stage') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/skaftab-in/simple-devops-website.git']])
            }
        }
        stage('Testing code') {
            steps {
                sh 'echo I am scanning the code'
                sh 'npm install mocha --save-dev'
                sh 'npm test'
            }
        }
        stage('Building Container') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }
        stage("Pushing-to-Docker(AWS-ECR)"){
            steps{
                sh """
                aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/x9f9n9e7
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        stage("Deploy-on-Kubernetes(AWS-EKS)"){
            steps{
                script {
                    sh """ 
                      aws eks update-kubeconfig --region ${AWS_REGION} --name K8s_for_aws-cicd-pipeline
                      if kubectl get deployment ${K8S_DEPLOYMENT}; then
                       kubectl set image deployment/${K8S_DEPLOYMENT} node-app-devops-project=${IMAGE_NAME}:${IMAGE_TAG} --record
                      else
                        kubectl apply -f app.yaml
                      fi
                        kubectl rollout status deployment/${K8S_DEPLOYMENT}
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Pipeline completed successfully. The application is deployed in production.'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}
```

---
#  🚧 🛠️🛑I am Under construction 🚧
---



---

## 🚀 How to Run the Pipeline Locally

To set up and run the pipeline locally, follow these steps:

1. **Clone the repository:**
    ```bash
    git clone https://github.com/skaftab-in/simple-devops-website.git
    ```
2. **Install Jenkins & Dependencies:** Ensure Jenkins is installed along with necessary plugins for Docker, AWS CLI, and Kubernetes.
3. **Set up AWS CLI:** Configure AWS CLI with the correct credentials:
    ```bash
    aws configure
    ```
4. **Set up EKS and ECR:** Make sure your **AWS EKS** and **ECR** are properly configured.

---

## 💬 Conclusion

This project demonstrates the power of modern CI/CD tools like **Jenkins**, **Docker**, **AWS ECR**, and **AWS EKS**. The fully automated pipeline ensures that updates are pushed, tested, and deployed with zero downtime. Additionally, the pipeline makes use of **Kubernetes'** rolling updates and scaling features, ensuring high availability and the ability to roll back in case of issues. This setup provides a seamless, highly available, and scalable solution for deploying applications in production.

---

### 🌐 GitHub Repository: [aws-cicd-pipeline-eks-ecr](https://github.com/skaftab-in/aws-cicd-pipeline-eks-ecr)
