pipeline {
      agent {
        kubernetes {
        label 'build-agent'
        defaultContainer 'kubectl'
        yamlFile 'build-pod.yaml'
     } 
  }
    environment{
        DOCKER_TAG = getDockerTag()
        IMAGE_TAG = "${env.BUILD_NUMBER}" 
        IMAGE_URL_WITH_TAG = "opeomotayo/node-app:${DOCKER_TAG}"
    }
    options {
      buildDiscarder(logRotator(numToKeepStr: '3'))
   }

    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t ${IMAGE_URL_WITH_TAG}"
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'dockerHub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u admin -p ${dockerHubPwd}"
                    sh "docker push ${IMAGE_URL_WITH_TAG}"
                }
            }
        }
        stage('Deploy to K8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['devops-server']) {
                    withCredentials([string(credentialsId: 'nexus-pwd', variable: 'nexusPwd')]) {
                        sh "scp -o StrictHostKeyChecking=no service.yaml deployment.yaml ope@49.13.238.19:/home/ope"
                        sh "ssh ope@49.13.238.19 kubectl apply -f ."
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
