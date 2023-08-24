pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout (time: 60, unit: 'MINUTES')
        timestamps()
  }
    
    environment {
		DOCKERHUB_CREDENTIALS = credentials('dockerhub-s4arnold')
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
                    docker build -t devopseasylearning/s4-arnold-auth:${BUILD_NUMBER} .
                    cd -
                 '''   
            }
        }
        
        stage('push auth') {
            steps {
                sh '''
                       docker push devopseasylearning/s4-arnold-auth:${BUILD_NUMBER} 
                 '''   
            }
        }      
    
        stage('Build ui') {
            steps {
                sh '''
                    cd UI
                    docker build -t devopseasylearning/s4-arnold-ui:${BUILD_NUMBER} .
                    cd -
                 '''   
            }
        }
        
        stage('push ui') {
            steps {
                    sh '''
                    docker push devopseasylearning/s4-arnold-ui:${BUILD_NUMBER} 
                 '''   
            }
        }       
        
        stage('Build db') {
            steps {
                sh '''
                      cd DB
                      docker build -t devopseasylearning/s4-arnold-db:${BUILD_NUMBER} .
                      cd -
                 '''   
            }
        }
        
        stage('push db') {
            steps {
                sh '''
                      docker push devopseasylearning/s4-arnold-db:${BUILD_NUMBER}                
                 '''   
            }
        }      
    
        stage('Build weather') {
            steps {
                sh '''
                      cd weather
                      docker build -t devopseasylearning/s4-arnold-weather:${BUILD_NUMBER} .
                      cd -
                 '''   
            }
        }
        stage('push weather') {
            steps {
                sh '''
                  docker push devopseasylearning/s4-arnold-weather:${BUILD_NUMBER}
             '''   
            }
        } 

        stage('Update charts') {
            steps {
                script {
                sh '''

git clone git@github.com:s4arnold/projects-charts.git
cd projects-charts

cat << EOF > charts/weatherapp-auth/dev-values.yaml
image:
  repository: devopseasylearning/s4-arnold-auth
  tag: ${BUILD_NUMBER}
EOF

cat << EOF > charts/weatherapp-mysql/dev-values.yaml
image:
  repository: devopseasylearning/s4-arnold-db
  tag: ${BUILD_NUMBER}
EOF

cat << EOF > charts/weatherapp-ui/dev-values.yaml
image:
  repository: devopseasylearning/s4-arnold-ui
  tag: ${BUILD_NUMBER}
EOF

cat << EOF > charts/weatherapp-weather/dev-values.yaml
image:
  repository: devopseasylearning/s4-arnold-weather
  tag: ${BUILD_NUMBER}
EOF

git config --global user.name "devopseasylearning"
git config --global user.email "tchuamarnold211@gmail.com"

git add -A
git commit -m "changes"
git push
 
             '''  
                }
            }
        }    
    }    
}

post {
   
   success {
      slackSend (channel: '#development-alerts', color: 'good', message: "SUCCESSFUL:  Application S4-PIPELINE  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

 
    unstable {
      slackSend (channel: '#development-alerts', color: 'warning', message: "UNSTABLE:  Application S4-PIPELINE  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    failure {
      slackSend (channel: '#development-alerts', color: '#FF0000', message: "FAILURE:  Application S4-PIPELINES Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
   
    cleanup {
      deleteDir()
    }
}



     
