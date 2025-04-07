pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        AWS_DOCKER_REGISTRY = '091912642030.dkr.ecr.us-east-2.amazonaws.com'
        APP_NAME = 'my_new_image'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:22-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('Build Docker Image') {
            agent any
            steps {
                withCredentials([usernamePassword(credentialsId: 'myNewUser', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:latest .
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args '-u root --entrypoint=""'
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'myNewUser', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        yum install jq -y || true
                        aws --version

                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster my-new-react-app-clusterprod --service my-new-react-app-Serviceprod --task-definition my-new-TaskDefinition-Prod:$LATEST_TD_REVISION
                    '''
                }
            }
        }
    }
}
