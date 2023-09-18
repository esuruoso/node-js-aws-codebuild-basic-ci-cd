pipeline {
  environment {
    imagename = "oesuruoso/node-web-app"
    registryCredential = 'docker_hub_login'
    //registryCredential = 'docker'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git([url: 'https://github.com/esuruoso/node-js-aws-codebuild-basic-ci-cd.git', branch: 'main', credentialsId: 'github_login'])

      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build imagename
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push("$BUILD_NUMBER")
             dockerImage.push('latest')

          }
        }
      }
    }
    stage('Deploy to K8s') {
      agent k8s-AWS-node-1
      steps{
        script {
          sh "sed -i 's,TEST_IMAGE_NAME,oesuruoso/node-web-app:$BUILD_NUMBER,' deployment.yaml"
          sh "cat deployment.yaml"
          sh "kubectl --kubeconfig=/home/ec2-user/config get pods"
          sh "kubectl --kubeconfig=/home/ec2-user/config apply -f deployment.yaml"
        }
      }
    }
    
    
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $imagename:$BUILD_NUMBER"
         sh "docker rmi $imagename:latest"

      }
    }
  }
}
