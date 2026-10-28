pipeline {
    agent any

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET = 'rateb-mock'
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
                        echo "Hello S3!" >> index.html
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
            }
        }
    }
}
