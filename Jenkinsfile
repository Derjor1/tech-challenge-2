pipeline {
    agent any

    environment {
        AWS_REGION   = 'us-east-1'
        AWS_ACCOUNT  = '883627150323'
        ECR_REPO     = 'tech-challenge-app'
        IMAGE_TAG    = "${BUILD_NUMBER}"
        IMAGE_URI    = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        KUBE_CONTEXT = 'arn:aws:eks:us-east-1:883627150323:cluster/tech-challenge-cluster'
        AWS_PAGER = ""
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    aws --version
                    kubectl version --client
                    helm version
                    docker version
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login \
                    --username AWS \
                    --password-stdin ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Ensure Buildx') {
            steps {
                sh '''
                    docker buildx inspect jenkins-builder >/dev/null 2>&1 || docker buildx create --name jenkins-builder --use
                    docker buildx use jenkins-builder
                    docker buildx inspect --bootstrap
                '''
            }
        }

        stage('Build and Push Image') {
            steps {
                sh '''
                    docker buildx build \
                      --platform linux/amd64 \
                      -t ${IMAGE_URI}:${IMAGE_TAG} \
                      -t ${IMAGE_URI}:latest \
                      --push .
                '''
            }
        }

        stage('Deploy to EKS with Helm') {
            steps {
                sh '''

                    helm upgrade --install tech-challenge-app ./helm/tech-challenge-app \
                      --set image.repository=${IMAGE_URI} \
                      --set image.tag=${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get svc
                    kubectl get ingress
                    kubectl get hpa
                '''
            }
        }
    }
}