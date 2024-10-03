pipeline {
    agent { label 'jenkins-agent' } // Jenkins agent or node label

    tools {
        jdk 'Java17'      // Ensure Java17 is configured in Jenkins
        maven 'Maven3'    // Ensure Maven3 is configured in Jenkins
    }

    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        AWS_REGION = "ap-south-1"  // Change to your AWS region
        AWS_ACCOUNT_ID = "9750-5033-4111" // Replace with your AWS Account ID
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        ECS_CLUSTER = "register-app-cluster" // Replace with your ECS Cluster name
        ECS_SERVICE = "register-app-service" // Replace with your ECS Service name
        CREDENTIALS_ID = "aws-credentials" // Make sure AWS credentials are stored in Jenkins with this ID
    }

    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'master', credentialsId: 'github', url: 'https://github.com/chinmay-2402/register-app.git'
            }
        }

        stage('Build Application') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Test Application') {
            steps {
                sh "mvn test"
            }
        }

        stage('Docker Build and Push to ECR') {
            steps {
                script {
                    // Authenticate to AWS ECR
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${CREDENTIALS_ID}"]]) {
                        sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker build -t ${APP_NAME}:${IMAGE_TAG} .
                        docker tag ${APP_NAME}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                        '''
                    }
                }
            }
        }

        stage('Update ECS Task Definition') {
            steps {
                script {
                    sh '''
                    aws ecs register-task-definition \
                    --family ${APP_NAME} \
                    --container-definitions "[{
                        \\"name\\": \\"${APP_NAME}\\", 
                        \\"image\\": \\"${ECR_REPO}:${IMAGE_TAG}\\", 
                        \\"memory\\": 512, 
                        \\"cpu\\": 256, 
                        \\"essential\\": true, 
                        \\"portMappings\\": [{
                            \\"containerPort\\": 80, 
                            \\"hostPort\\": 80
                        }]
                    }]"
                    '''
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    sh '''
                    aws ecs update-service \
                    --cluster ${ECS_CLUSTER} \
                    --service ${ECS_SERVICE} \
                    --force-new-deployment
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace after build.'
            cleanWs()
        }
        success {
            echo 'Build and deployment were successful!'
        }
        failure {
            echo 'Build or deployment failed. Please check the logs.'
        }
    }
}

