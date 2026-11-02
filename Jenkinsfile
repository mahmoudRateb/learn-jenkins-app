pipeline {
    agent {
        docker {
            image 'amazon/aws-cli'
            args "--entrypoint=''"
        }
    }
    environment {
        AWS_S3_BUCKET = 'rateb-mock'
        AWS_REGION = 'eu-central-1'
    }

    stages {
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
                        aws ecs register-task-definition  \
                        --cli-input-json file://aws/task-definition-prod.json \
                        --region $AWS_REGION
                    '''
                }
            }
        }
    }
}
