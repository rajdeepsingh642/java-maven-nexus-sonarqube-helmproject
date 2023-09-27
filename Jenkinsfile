pipeline{
    agent any

     tools { 
      maven  'mymaven'
     }

    stages{
         stage('checkout'){
             steps{
                git 'https://github.com/rajdeepsingh642/java-maven-nexus-sonarqube-helmproject.git'
             }
         }
                
        
        stage('sonar quality check'){
            steps{
                script{
                withSonarQubeEnv(credentialsId: 'sonar_qube') {
                   sh "mvn clean install sonar:sonar"
                 }
               }
            }
        }
        
         stage('docker build image'){
            steps{
                script{
                    sh 'docker build -t 192.168.1.228:8083/java-app:${BUILD_ID} .'

                }
            }
        }
        stage('docker image push nexus'){
            steps{
                script{
                     withCredentials([string(credentialsId: 'nexus-token', variable: 'nexus_creds')]) {
                        sh "docker login -u admin -p $nexus_creds  192.168.1.228:8083"
                        sh 'docker push 192.168.1.228:8083/java-app:${BUILD_ID}'

                }
          
                  }
                    }
        }


           stage('check datree'){
          steps{
            script{
              
                    dir(' helm/java') {
                      withEnv(['DATREE_TOKEN=aa97d52e-a99b-4982-bcf4-e672a8b95db6']) {
                   sh 'helm datree test .' 
              }
                 }

            }
          }
          }





         stage('pushin helm chart to nexus'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus-token', variable: 'nexus_creds')]) {
             
    
                     echo "Packing helm chart"
                      sh "helm package -d ${WORKSPACE}/helm ${WORKSPACE}/helm/java"
           // sh "${WORKSPACE}/build.sh --pack_helm --push_helm --helm_repo ${HELM_REPO} --helm_usr ${HELM_USR} --helm_psw ${HELM_PSW}"
                        sh "curl -u admin:${nexus_creds} http://192.168.1.228:8081/repository/helm-repo/ --upload-file ${WORKSPACE}/helm/java-1.tgz -v"
            
        }
                }
            }
         }
    }
}
       
        }

    }

      stage('deploye to k8s'){
          steps{
            script{
               
                     dir(' helm/java') {
                        'sh helm upgrade --install --set image.repository="192.168.1.226:8083/java-app" --set image.tag="$BUILD_ID" myjavaaap singh/'
                     

            }
          }
       } 
    }

    }
}  