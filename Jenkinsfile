pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout (time: 60, unit: 'MINUTES')
        timestamps()
  }
    
    environment {
		DOCKERHUB_CREDENTIALS = credentials('dockerhub-s4-pipeline')
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

        stage('Dockerhub Login') {
	    	steps {
	    		sh '''
                    echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                '''
	    	}
	    }
    
        
        stage('Build auth') { 
            steps {
                sh '''
                    cd auth
                    docker build -t devopseasylearning/s4-pipeline-auth:${BUILD_NUMBER} .
                    cd -
                 '''   
            }
        }
        
        stage('push auth') {
            steps {
                sh '''
                       docker build -t devopseasylearning/s4-pipeline-auth:${BUILD_NUMBER} 
                 '''   
            }
        }      
    
        stage('Build ui') {
            steps {
                sh '''
                    cd UI
                    docker build -t devopseasylearning/s4-pipeline-ui:${BUILD_NUMBER} .
                    cd -
                 '''   
            }
        }
        
        stage('push ui') {
            steps {
                    sh '''
                    docker build -t devopseasylearning/s4-pipeline-ui:${BUILD_NUMBER} 
                 '''   
            }
        }       
        
        stage('Build db') {
            steps {
                sh '''
                      cd DB
                      docker build -t devopseasylearning/s4-pipeline-db:${BUILD_NUMBER} .
                      cd -
                 '''   
            }
        }
        
        stage('push db') {
            steps {
                sh '''
                      docker build -t devopseasylearning/s4-pipeline-db:${BUILD_NUMBER}                
                 '''   
            }
        }      
    
        stage('Build weather') {
            steps {
                sh '''
                      cd weather
                      docker build -t devopseasylearning/s4-pipeline-weather:${BUILD_NUMBER} .
                      cd -
                 '''   
            }
        }
        stage('push weather') {
            steps {
                sh '''
                  docker build -t devopseasylearning/s4-pipeline-weather:${BUILD_NUMBER}
             '''   
            }
        }   
    }    
}

post {
   
   success {
      slackSend (channel: '#outage', color: 'good', message: "SUCCESSFUL:  Application S4-PIPELINE  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

 
    unstable {
      slackSend (channel: '#outage', color: 'warning', message: "UNSTABLE:  Application S4-PIPELINE  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    failure {
      slackSend (channel: '#outage', color: '#FF0000', message: "FAILURE:  Application S4-PIPELINES Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
   
    cleanup {
      deleteDir()
    }
}



     
