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
                sh 'docker stop $(docker ps -q) || true'
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
                script {
                    if (env.GIT_BRANCH == 'origin/main'){
                        sh 'docker run -d --name lbg-test-container -p 80:5500 -e PORT=5500 parilvadher/lbg-python:latest'
                        sh 'sleep 3'
                        sh 'python3 lbg.test.py'
                        sh 'docker stop lbg-test-container'                        
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh 'docker run -d --name lbg-test-container-dev -p 80:5500 -e PORT=5500 parilvadher/lbg-python-dev:latest'
                        sh 'sleep 3'
                        sh 'python3 lbg.test.py'
                        sh 'docker stop lbg-test-container-dev'                        
                    }
                }
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
