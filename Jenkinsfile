pipeline {
    agent any   
    stages {       
        stage('Init') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        kubectl create ns production || echo "------- Production Namespace Already Exists -------"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl create ns development || echo "------- Development Namespace Already Exists -------"
                        '''
                    } else {
                        sh'echo "Unknown branch"'
                    }
                }
            }
        }
        stage('Clean up previous artifacts') {
            steps {
                sh 'docker rm -f $(docker ps -aq) || true'
                sh 'docker rmi -f $(docker images) || true'
           }
        }       
        stage('Build Image') {            
           steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main'){
                        sh '''
                        docker build -t parilvadher/lbg-python:latest -t parilvadher/lbg-python:v${BUILD_NUMBER} .
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker build -t parilvadher/lbg-python-dev:latest -t parilvadher/lbg-python-dev:v${BUILD_NUMBER} .
                        '''
                    }
                }
            } 
        }
        stage('Run Tests') {
            steps {
                sh 'docker run -d -p 80:8080 -e PORT=8080 parilvadher/lbg-python:latest'
                sh 'python3 lbg.test.py'
            }
        }
        stage('Push Images') {
            steps {

                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        docker push parilvadher/lbg-python:latest
                        docker push parilvadher/lbg-python:v${BUILD_NUMBER}
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker push parilvadher/lbg-python-dev:latest
                        docker push parilvadher/lbg-python-dev:v${BUILD_NUMBER}
                        '''
                    } else {
                        sh'echo "Unknown branch"'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh'''
                        kubectl apply -f ./kubernetes -n production
                        kubectl set image deployment/flask-deployment flask-container=parilvadher/lbg-python:v${BUILD_NUMBER} -n production
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh'''
                        kubectl apply -f ./kubernetes -n development
                        kubectl set image deployment/flask-deployment flask-container=parilvadher/lbg-python-dev:v${BUILD_NUMBER} -n development
                        '''
                    } else {
                        sh'echo "Unknown branch"'
                    }
                }
            }
        }       
    }   
}
