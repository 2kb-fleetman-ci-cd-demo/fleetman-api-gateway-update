pipeline {
   agent any

   environment {
     // You must set the following environment variables
     // ORGANIZATION_NAME
     // YOUR_DOCKERHUB_USERNAME (it doesn't matter if you don't have one)

     SERVICE_NAME = "fleetman-api-gateway-update"
     REPOSITORY_TAG = "${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
     DOCKER = credentials('DockerHub')
   }

   stages {
      stage('Preparation') {
         steps {
             cleanWs()
             git credentialsId: 'GitHub', url: "https://github.com/${ORGANIZATION_NAME}/${SERVICE_NAME}"
         }
      }
      stage('Build') {
         steps {
             sh '''mvn clean package'''
         }
      }

      stage('Build Image') {
         steps {
             sh 'docker image build -t ${REPOSITORY_TAG} .'
         }
      }

      stage('Logging to docker hub') {
          steps {
              sh 'docker login -u ${YOUR_DOCKERHUB_USERNAME} -p ${DOCKER}'
          }
      }

      stage('Pushing Image to docker hub') {
          steps {
              sh 'docker push ${REPOSITORY_TAG}'
          }
      }

      stage('Deploy to Cluster') {
          steps {
              sh 'envsubst < ${WORKSPACE}/deploy.yaml'
              withKubeConfig([credentialsId: 'k8s-new', serverUrl: 'https://192.168.1.45:6443']) {
                  sh 'kubectl apply -f -'
              }
          }
      }
   }
}
