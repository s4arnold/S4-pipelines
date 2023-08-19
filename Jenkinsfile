pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from the repository
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                // Run build commands here
                sh 'make build'
            }
        }
        
        stage('Test') {
            steps {
                // Run unit tests here
                sh 'make test'
            }
        }
        
        stage('Deploy') {
            steps {
                // Deploy your application here
                sh 'make deploy'
            }
        }
    }
}
