pipeline {
    agent { label 'jenkins-agent' }
    tools {
        jdk 'Java17'      // Make sure "Java17" is defined in Jenkins Global Tool Configuration
        maven 'Maven3'    // Make sure "Maven3" is defined in Jenkins Global Tool Configuration
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "chinmay2402"
        DOCKER_PASS = 'dockerhub'    // Make sure "dockerhub" is defined as a secret text credential in Jenkins
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'master', credentialsId: 'github', url: 'https://github.com/chinmay-2402/register-app.git'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }
        
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub') { // Using the credentialsId for Docker Hub authentication
                        def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        
                        // Pushing with the versioned tag
                        dockerImage.push("${IMAGE_TAG}")
                        // Pushing with the 'latest' tag
                        dockerImage.push("latest")
                    }
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
            echo 'Build was successful!'
        }
        failure {
            echo 'Build failed. Please check the logs.'
        }
    }
}

