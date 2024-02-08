pipeline {
   agent none
  environment{
      BUILD_SERVER_IP='ec2-user@172.31.37.103'
       IMAGE_NAME='theetla/java-mvn-cicdrepos:php$BUILD_NUMBER'
       DEPLOY_SERVER_IP='ec2-user@172.31.32.116'
   }
    stages {          
        stage('BUILD DOCKERIMAGE AND PUSH TO DOCKERHUB') {
            agent any            
            steps {
                script{
                sshagent(['slave1']) {
                withCredentials([usernamePassword(credentialsId: 'buildserver', passwordVariable: 'mydockerhubpassword', usernameVariable: 'mydockerhubusername')]) {
                echo "Packaging the apps"
                sh "scp -o StrictHostKeyChecking=no -r docker-files ${BUILD_SERVER_IP}:/home/ec2-user"
                sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER_IP} 'bash ~/docker-files/docker-script.sh'"
                sh "ssh ${BUILD_SERVER_IP} sudo docker build -t ${IMAGE_NAME} /home/ec2-user/docker-files/"
                sh "ssh ${BUILD_SERVER_IP} sudo docker login -u $mydockerhubusername -p $mydockerhubpassword"
                sh "ssh ${BUILD_SERVER_IP} sudo docker push ${IMAGE_NAME}"
              }
            }
            }
        }
        }
       stage('DEPLOY DOCKER CONATINER USING DOCKER_COMPOSE'){
           agent any
           steps{
               script{
                    sshagent(['slave1']){
                         withCredentials([usernamePassword(credentialsId: 'buildserver', passwordVariable: 'mydockerhubpassword', usernameVariable: 'mydockerhubusername')]) {
                         sh "scp -o StrictHostKeyChecking=no -r docker-files ${DEPLOY_SERVER_IP}:/home/ec2-user"
                         sh "ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER_IP} 'bash ~/docker-files/docker-script.sh'"
                         sh "ssh ${DEPLOY_SERVER_IP} sudo docker login -u $mydockerhubusername -p $mydockerhubpassword"
                         sh "ssh ${DEPLOY_SERVER_IP} bash /home/ec2-user/docker-files/docker-compose-script.sh ${IMAGE_NAME}"
                         }
                    }
               }
           }
       }
    }
}
