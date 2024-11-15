pipeline {
    agent any
    environment {
        DOCKER_IMAGE="lbg"
        DOCKER_HUB_PAT=credentials('DOCKER_HUB_PAT')
        PORT=5001
    }
    stages {
        stage('Clean up previous artifacts') {
            steps {
                sh 'docker rm -f $(docker ps -aq) || true'
                sh 'docker rmi -f $(docker images) || true'
           }
        }        
        stage('Build Image') {
            steps {
                sh 'export PORT=${PORT}'
                sh 'docker build -t ${DOCKER_HUB_PAT_USR}/${DOCKER_IMAGE}'
           }
        }
        stage('Run Container') {
            steps {
                sh 'docker run -d -p 80:$PORT -e PORT=${PORT} ${DOCKER_IMAGE}'
            }
        }
    }   
}
