# Jenkinsfile

pipeline {
    agent any

    environment {
        TELEGRAM_TOKEN_PROD = credentials('TelegramTokenProd')
        CHAT_ID_PROD = credentials('ChatIDProd')
    }

    stages {
        stage('build docker image') {
            steps {
                sh 'ssh -o StrictHostKeyChecking=no itprphirak@172.19.24.33 "cd /apps/Dummy-Bank/api-v1;\
                git pull;\
                sudo docker build -t bank-api .;\
                "'
            }
        }
        stage('push image to private registry'){
             steps {
                sh 'ssh -o StrictHostKeyChecking=no itprphirak@172.19.24.33 "cd /apps/Dummy-Bank/api-v1;\
                sudo docker login 172.19.24.47:5000;\
                sudo docker tag bank-api 172.19.24.47:5000/bank-api:${BUILD_NUMBER};\
                sudo docker push 172.19.24.47:5000/bank-api:${BUILD_NUMBER};\
                "'
            }
        }
        stage('deploy to dev'){
            steps {
                sh 'ssh -o StrictHostKeyChecking=no itprphirak@172.19.24.33 "cd /apps/Dummy-Bank/api-v1;\
                sudo docker login 172.19.24.47:5000;\
                sudo docker service rm dummy-bank_api;\
                sudo TAG=${BUILD_NUMBER} docker stack deploy --compose-file docker-compose-stack-deploy.yml dummy-bank;\
                "'
            }
        }
    }

}
