pipeline {
    agent { label 'master' }
    tools {
        jdk 'Java17'      // Ensure "Java17" is defined in Jenkins Global Tool Configuration
        maven 'Maven3'    // Ensure "Maven3" is defined in Jenkins Global Tool Configuration
    }
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "chinmay2402"
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
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') { // Use credentialsId for Docker authentication
                        def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        
                        // Push with the versioned tag
                        dockerImage.push("${IMAGE_TAG}")
                        // Push with the 'latest' tag
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

