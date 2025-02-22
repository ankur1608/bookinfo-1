pipeline {
environment {
registry = "docker.io/ankur1308/bookinfo_cicd"
registryCredential = 'ankur_dockerhub_cred'
dockerImage = ''
}
agent any
stages {
stage('Build Image') {
steps{
dir('src/productpage') {
script {
sh """
IMAGE="$registry:$BUILD_NUMBER" 
sudo podman build -t \${IMAGE} . 
"""

}}
}
}

stage ("sonar-analysis"){
steps {
dir('src/productpage') {
withSonarQubeEnv('sonar_scanner') {
echo "===========Performing Sonar Scan============"
sh "sonar-scanner \
  -Dsonar.projectKey=Bookinfo \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://jenkins-server.10.25.55.71.nip.io:9000 \
  -Dsonar.login=bookinfo"
}
}
}
}
stage('Push Image to registry') {
steps{
script {
docker.withRegistry( '', registryCredential ) {
// dockerImage.push()
                    sh """
                    #!/bin/bash

                    # Construct Image Name
                    IMAGE="$registry:$BUILD_NUMBER" 

                    sudo podman push \${IMAGE} --tls-verify=false
                    """
}
}
}
}
stage('App deploy') {
steps{
sh "kubectl --kubeconfig /var/lib/jenkins/k3s_config set image deployment productpage-v1 productpage=$registry:$BUILD_NUMBER"
}
}   
stage('Cleaning up') {
steps{
sh "sudo podman rmi $registry:$BUILD_NUMBER"
}
}
}
}
