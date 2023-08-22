pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout (time: 60, unit: 'MINUTES')
        timestamps()
  }
    
    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub-s4arnold')
	}
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
        
        stages {

    stage('Dockerhub Login') {
		steps {
			sh '''
                echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -username "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
            '''
		}
	}

        stage('Hello') {
            steps {
                sh '''
                ls 
                pwd
                '''
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

     
