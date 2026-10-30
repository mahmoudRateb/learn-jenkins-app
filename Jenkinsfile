pipeline {
    agent {
        docker {
            image 'amazon/aws-cli'
            args "--entrypoint=''"
        }
    }
    environment {
        AWS_S3_BUCKET = 'rateb-mock'
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
                        aws s3 sync build s3://$AWS_S3_BUCKET
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
                        aws ecs register-task-definition  --cli-input-json file://aws/task-definition-prod.json
                    '''
                }
            }
        }
    }
}
