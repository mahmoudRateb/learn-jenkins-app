pipeline {
    agent any

    environment {
        AWS_S3_BUCKET = 'rateb-mock'
        AWS_REGION = 'eu-central-1'
        AWS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
        AWS_SERVICE = 'LearnJenkinsApp-Service-Prod'
        AWS_TASK_DEF = 'LearnJenkinsApp-TaskDef-Prod'
    }

    stages {
        stage('Build Docker image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                sh '''
                   docker build -t myjenkinsapp .
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    args "-u root --entrypoint=''"
                }
            }
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
