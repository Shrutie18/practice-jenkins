pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'  // Update this to your AWS region
        ECR_REPO_NAME = 'my-app-repo'  // Use the name of your ECR repository
        AWS_ACCOUNT_ID = '717279723360' // Your AWS Account ID
        IMAGE_TAG = "${env.BUILD_NUMBER}"  // Build number for image tag
        CLUSTER_NAME = 'my-ecs-cluster' // ECS Cluster name
        SERVICE_NAME = 'my-app-service' // ECS Service name
        TASK_DEFINITION_FAMILY = 'taskdefinition' // Task Definition Family
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm  // Gets the code from your source control (like Git)
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    dockerImage = docker.build("${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    // Log into ECR
                    sh """
                    \$(aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com)
                    """
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    // Push the Docker image to ECR
                    dockerImage.push()
                }
            }
        }

        stage('Update ECS Service') {
            steps {
                script {
                    // Register a new ECS task with the new image
                    def taskDefinition = """
                    {
                        "family": "${TASK_DEFINITION_FAMILY}",
                        "containerDefinitions": [
                            {
                                "name": "container",
                                "image": "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}",
                                "essential": true,
                                "memory": 13312,
                                "cpu": 3072,
                                "portMappings": [
                                    {
                                        "containerPort": 8080,
                                        "hostPort": 8080
                                    }
                                ]
                            }
                        ]
                    }
                    """
                    
                    writeFile(file: 'task-definition.json', text: taskDefinition)
                    
                    // Update the ECS service to use the new task
                    sh """
                    aws ecs register-task-definition --cli-input-json file://task-definition.json
                    aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --force-new-deployment
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment to ECS Fargate was successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
