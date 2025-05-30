pipeline {
    agent any

    // environment {
    //     AWS_DOCKER_REGISTRY = '612634926349.dkr.ecr.us-east-2.amazonaws.com'
    //     APP_NAME = 'my_new_image'
    //     AWS_DEFAULT_REGION = 'us-east-2'
    // }

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

        // stage('Build and Push Docker Image') {
        //     agent {
        //         docker {
        //             image 'public.ecr.aws/aws-cli/aws-cli:2.13.14-docker'
        //             reuseNode true
        //             args '-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
        //         }
        //     }
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'myNewUser', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
        //             sh '''
        //                 echo "Logging into ECR..."
        //                 aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY

        //                 echo "Building Docker image..."
        //                 docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME .

        //                 echo "Pushing Docker image..."
        //                 docker push $AWS_DOCKER_REGISTRY/$APP_NAME:latest
        //             '''
        //         }
        //     }
        // }

      stage('AWS S3 Upload') {
    agent {
        docker {
            image 'amazon/aws-cli'
            reuseNode true
            args '--entrypoint=""'
        }
    }
    environment {
        AWS_S3_BUCKET = 'temp-20250405'
    }
    steps {
        withCredentials([usernamePassword(credentialsId: 'my-temp', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
            sh '''
                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                export AWS_DEFAULT_REGION=us-east-2

                aws --version
                aws s3 ls

                echo "Hello S3!" > index.html
                # aws s3 cp index.html s3://temp-20250405/index.html
                aws s3 sync build s3://$AWS_S3_BUCKET
            '''
        }
    }
}


        }
    }

