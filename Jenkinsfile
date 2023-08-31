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
        stage('Setup parameters') {
            steps {
                script {
                    properties([
                        parameters([
                            choice( 
                                choices: ['DEV', 'QA', 'PREPOD'], 
                                name: 'ENVIRONMENT'
                            ),
                            string(
                                defaultValue: ' ',
                                name: 'auth_tag',
                                description: '''type the auth image tag''',
                                ),
                            
                            string(
                                defaultValue: ' ',
                                name: 'ui_tag',
                                description: '''type the weather image tag''',
                                ),
                            
                            string(
                                defaultValue: ' ',
                                name: 'db_tag',
                                description: '''type the ui image tag''',
                                ),
                            
                            string(
                                defaultValue: ' ',
                                name: 'weather_tag',
                                description: '''type the db image tag''',
                                ),    
                            
                            string(name: 'WARNTIME',
                            defaultValue: '0',
                            description: '''Warning time (in minutes) before starting upgrade'''),

                            string(
                                defaultValue: 'develop',
                                name: 'Please_leave_this_section_as_it_is',
                            ),
                        ])
                    ])
                }
            }
        }


        stage('warning') {
              steps {
                script {
                    notifyUpgrade(currentBuild.currentResult, "WARNING")
                    sleep(time:env.WARNTIME, unit:"MINUTES")
                }
            }
        }
        stage('SonarQube analysis') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
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
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                    cd auth
                    docker build -t devopseasylearning/s4-arnold-auth:${BUILD_NUMBER} .
                    cd -
                 '''   
            }
        }
        
        stage('push auth') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                       docker push devopseasylearning/s4-arnold-auth:${BUILD_NUMBER} 
                 '''   
            }
        }      
    
        stage('Build ui') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                    cd UI
                    docker build -t devopseasylearning/s4-arnold-ui:${BUILD_NUMBER} .
                    cd -
                 '''   
            }
        }
        
        stage('push ui') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                    sh '''
                    docker push devopseasylearning/s4-arnold-ui:${BUILD_NUMBER} 
                 '''   
            }
        }       
        
        stage('Build db') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                      cd DB
                      docker build -t devopseasylearning/s4-arnold-db:${BUILD_NUMBER} .
                      cd -
                 '''   
            }
        }
        
        stage('push db') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                      docker push devopseasylearning/s4-arnold-db:${BUILD_NUMBER}                
                 '''   
            }
        }      
    
        stage('Build weather') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                      cd weather
                      docker build -t devopseasylearning/s4-arnold-weather:${BUILD_NUMBER} .
                      cd -
                 '''   
            }
        }
        stage('push weather') {
                when{
                   expression {
                     env.ENVIRONMENT == 'DEV' }
                }
            steps {
                sh '''
                  docker push devopseasylearning/s4-arnold-weather:${BUILD_NUMBER}
             '''   
            }
        } 

        stage('QA: pull images') {
            when{
                expression {
                     env.ENVIRONMENT == 'QA' }
                }
            steps {
                sh '''
                   docker pull  devopseasylearning/s4-arnold-auth:$auth_tag 
                   docker pull  devopseasylearning/s4-arnold-ui:$ui_tag 
                   docker pull  devopseasylearning/s4-arnold-db:$db_tag 
                   docker pull  devopseasylearning/s4-arnold-weather:$weather_tag 
                
            '''       
            }
        }

        stage('QA: tag images') {
            when{
                expression {
                    env.ENVIRONMENT == 'QA' }
                }
            steps {
                sh '''
                   docker tag  devopseasylearning/s4-arnold-auth:$auth_tag   devopseasylearning/s4-arnold-auth:qa-$auth_tag
                   docker tag  devopseasylearning/s4-arnold-ui:$ui_tag       devopseasylearning/s4-arnold-ui:qa-$ui_tag
                   docker tag  devopseasylearning/s4-arnold-db:$db_tag       devopseasylearning/s4-arnold-db:qa-$db_tag
                   docker tag  devopseasylearning/s4-arnold-weather:$weather_tag   devopseasylearning/s4-arnold-weather:qa-$weather_tag 
                
            '''       
        }
    }

        stage('Update DEV charts') {
            when{
                expression {
                   env.ENVIRONMENT == 'DEV' }
                }
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

git config --global user.name "s4arnold"
git config --global user.email "tchuamarnold211@gmail.com"

git add -A
git commit -m "changes"
git push
    
            '''  
                }
            }
        } 
        
        stage('Update QA charts') {
            when{
                expression {
                   env.ENVIRONMENT == 'QA' }
                }
            steps {
                script {
                
                  sh '''

git clone git@github.com:s4arnold/projects-charts.git
cd projects-charts

cat << EOF > charts/weatherapp-auth/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-auth
   tag: qa-$auth_tag
EOF

cat << EOF > charts/weatherapp-ui/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-ui
   tag: qa-$ui_tag
EOF

cat << EOF > charts/weatherapp-mysql/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-db
   tag: qa-$db_tag
EOF

cat << EOF > charts/weatherapp-weather/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-weather
   tag: qa-$weather_tag
EOF

git config --global user.name "s4arnold"
git config --global user.email "tchuamarnold211@gmail.com"

git add -A
git commit -m "changes"
git push

            '''  
                }
            }
        }
        
         stage('Update PREPROD charts') {
            when{
                expression {
                   env.ENVIRONMENT == 'PREPROD' }
                }
            steps {
                script {
                
                  sh '''

git clone git@github.com:s4arnold/projects-charts.git
cd projects-charts

cat << EOF > charts/weatherapp-auth/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-auth
   tag: ${BUILD_NUMBER}
EOF

cat << EOF > charts/weatherapp-ui/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-ui
   tag: ${BUILD_NUMBER}
EOF

cat << EOF > charts/weatherapp-mysql/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-db
   tag: ${BUILD_NUMBER}
EOF

cat << EOF > charts/weatherapp-weather/qa-values.yaml
image:
   repository: devopseasylearning/s4-arnold-weather
   tag: ${BUILD_NUMBER}
EOF

git config --global user.name "s4arnold"
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
    always {
      script {
        notifyUpgrade(currentBuild.currentResult, "POST")
      }
    }
    
}

def notifyUpgrade(String buildResult, String whereAt) {
  if (Please_leave_this_section_as_it_is == 'origin/develop') {
    channel = 'development-alerts'
  } else {
    channel = 'development-alerts'
  }
  if (buildResult == "SUCCESS") {
    switch(whereAt) {
      case 'WARNING':
        slackSend(channel: channel,
                color: "#439FE0",
                message: "S4-weather: Upgrade starting in ${env.WARNTIME} minutes @ ${env.BUILD_URL}  Application S4-weather")
        break
    case 'STARTING':
      slackSend(channel: channel,
                color: "good",
                message: "S4-weather: Starting upgrade @ ${env.BUILD_URL} Application S4-weather")
      break
    default:
        slackSend(channel: channel,
                color: "good",
                message: "S4-weather: Upgrade completed successfully @ ${env.BUILD_URL}  Application S4-weather")
        break
    }
  } else {
    slackSend(channel: channel,
              color: "danger",
              message: "S4-weather: Upgrade was not successful. Please investigate it immediately.  @ ${env.BUILD_URL}  Application S4-weather")
  }
}
 



     
