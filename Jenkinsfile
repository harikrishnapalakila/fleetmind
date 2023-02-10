pipeline {
    agent any
        environment {
        registry = "fleetmindacr"
        //- update your credentials ID after creating credentials for connecting to Docker Hub
        registryCredential = 'ACR'
        registryUrl = 'fleetmindacr.azurecr.io'
        dockerImage = ''
    }
    stages {

        stage ('checkout') {
            steps {
            checkout scmGit(branches: [[name: '*/dependabot/pip/flask-1.0']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/harikrishnapalakila/fleetmind.git']])
            }
        }
       
        stage ('Build docker image') {
            steps {
                script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
                //dockerImage = docker.build registry + ":$BUILD_NUMBER"

                }
            }
        }
       
         // Uploading Docker images into Docker Hub
    stage('Upload Image') {
     steps{   
         script {
             docker.withRegistry( 'http://${registryUrl}', registryCredential ) {
            dockerImage.push()
            }
        }
      }
    }

    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
   
    stage ('K8S Deploy') {
        steps {
            script {
                kubernetesDeploy(
                    configs: 'k8s-deployment.yaml',
                    kubeconfigId: 'K8S',
                    enableConfigSubstitution: true
                    )           
               
            }
        }
    }
  
    }  
}

always {
  // remove built docker image and prune system
  print 'Cleaning up the Docker system.'
  sh 'docker system prune -f'
}
