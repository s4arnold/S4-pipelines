pipeline {
    agent any
    
    stages {
        stage('SonarQube analysis') {
            agent {
                docker {
                  image 'sonarsource/sonar-scanner-cli:4.7.0'
                }
               }
               environment {
        CI = 'true'
        //  scannerHome = tool 'Sonar'
        scannerHome='/opt/sonar-scanner'
    }
            steps{
                withSonarQubeEnv('Sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
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
                sh '''
                pwd
                ls
                '''

            }
        }
    }
}
