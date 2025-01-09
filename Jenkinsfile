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
