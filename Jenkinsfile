pipeline {

  agent any
  environment {
        REGISTRY = 'ikhsannugs'
  }
    stages{
      stage('Build with Docker') {
        steps {
          sh "docker build -f ./backend/Dockerfile -t ${REGISTRY}/project-backend:${BRANCH_NAME}-${BUILD_NUMBER} ."
          sh "docker build -f ./mongo/Dockerfile -t ${REGISTRY}/project-mongo:${BRANCH_NAME}-${BUILD_NUMBER} ."
          sh "docker build -f ./frontend/Dockerfile -t ${REGISTRY}/project-frontend:${BRANCH_NAME}-${BUILD_NUMBER} ."
        }
      }
      stage('Publish Docker Image') {
        steps {
          sh "docker push ${REGISTRY}/project-backend:${BRANCH_NAME}-${BUILD_NUMBER} ."
          sh "docker push ${REGISTRY}/project-mongo:${BRANCH_NAME}-${BUILD_NUMBER} ."
          sh "docker push ${REGISTRY}/project-frontend:${BRANCH_NAME}-${BUILD_NUMBER} ."
        }
      }
      stage('Deploy to Kubernetes') {
        when {
           changelog 'deployment'
        }
        steps {
          script {
            if ( env.GIT_BRANCH == 'dev' ) {
              sh "sed -i 's/TAG-DB/${BRANCH_NAME}-${BUILD_NUMBER}/g' ./manifest/charts/mern/values.yaml"
              sh "sed -i 's/TAG-BACKEND/${BRANCH_NAME}-${BUILD_NUMBER}/g' ./manifest/charts/mern/values.yaml"
              sh "sed -i 's/TAG-FRONTEND/${BRANCH_NAME}-${BUILD_NUMBER}/g' ./manifest/charts/mern/values.yaml"
              sh "helm uninstall mern"
              sh "helm install mern ./manifest/charts/mern -n dev"
            }
            else if ( env.GIT_BRANCH == 'main' ) {
              sh "sed -i 's/TAG-DB/${BRANCH_NAME}-${BUILD_NUMBER}/g' ./manifest/charts/mern/values.yaml"
              sh "sed -i 's/TAG-BACKEND/${BRANCH_NAME}-${BUILD_NUMBER}/g' ./manifest/charts/mern/values.yaml"
              sh "sed -i 's/TAG-FRONTEND/${BRANCH_NAME}-${BUILD_NUMBER}/g' ./manifest/charts/mern/values.yaml"
              sh "helm uninstall mern"
              sh "helm install mern ./manifest/charts/mern -n prod"
            }
          }
        }
      }
    }
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir()
        }
        success {
            echo 'I succeeded!'
        }
        failure {
            echo 'I failed :('
        }
    }
}