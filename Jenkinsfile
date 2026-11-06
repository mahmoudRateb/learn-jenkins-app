pipeline {
    agent {
        docker {
            image 'amazon/aws-cli'
            args "-u root --entrypoint=''"
        }
    }
    environment {
        AWS_S3_BUCKET = 'rateb-mock'
        AWS_REGION = 'eu-central-1'
        AWS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
        AWS_SERVICE = 'LearnJenkinsApp-Service-Prod'
        AWS_TASK_DEF = 'LearnJenkinsApp-TaskDef-Prod'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
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
                    ls -la
                '''
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t myjenkinsapp .'
            }
        }
        
        stage('AWS') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws_creds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws --version
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Deploy to AWS') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws_creds',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_REVISION=$(aws ecs register-task-definition  \
                            --cli-input-json file://aws/task-definition-prod.json \
                            --region $AWS_REGION \
                            | jq '.taskDefinition.revision')
                        echo "The revision number is $LATEST_REVISION"
                        aws ecs update-service \
                            --cluster $AWS_CLUSTER \
                            --service $AWS_SERVICE \
                            --task-definition $AWS_TASK_DEF:$LATEST_REVISION
                        aws ecs wait services-stable \
                            --cluster $AWS_CLUSTER \
                            --services $AWS_SERVICE
                    '''
                }
            }
        }
    }
}
