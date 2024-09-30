pipeline {
    agent { label 'jenkins-agent' }
    tools {
        jdk 'Java17'      // Make sure "Java17" is defined in Jenkins Global Tool Configuration
        maven 'Maven3'    // Make sure "Maven3" is defined in Jenkins Global Tool Configuration
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
