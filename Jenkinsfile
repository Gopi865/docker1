pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
        NEXUS_URL  = "20.114.3.136:8080"
        IMAGE_URL_WITH_TAG = "${NEXUS_URL}/node-app:${DOCKER_TAG}"
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t ${IMAGE_URL_WITH_TAG}"
            }
        }
        stage('Nexus Push'){
            steps{
                withCredentials([string(credentialsId: 'nexus-pwd', variable: 'nexusPwd')]) {
                    sh "docker login -u admin -p ${nexusPwd} ${NEXUS_URL}"
                    sh "docker push ${IMAGE_URL_WITH_TAG}"
                }
            }
        }
        stage('Docker Deploy Dev'){
            steps{
                sshagent(['tomcat-dev']) {
                    withCredentials([string(credentialsId: 'nexus-pwd', variable: 'nexusPwd')]) {
                        sh "ssh azure@20.114.3.136 docker login -u admin -p ${nexusPwd} ${NEXUS_URL}"
                    }
					// Remove existing container, if container name does not exists still proceed with the build
					sh script: "ssh azure@20.114.3.136 docker rm -f nodeapp",  returnStatus: true
                    
                    sh "ssh azure@20.114.3.136 docker run -d -p 8080:8080 --name nodeapp ${IMAGE_URL_WITH_TAG}"
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
