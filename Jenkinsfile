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
                        yum install jq -y
                        LATEST_REVISION=$(aws ecs register-task-definition  \
                            --cli-input-json file://aws/task-definition-prod.json \
                            --region $AWS_REGION \
                            jq '.taskDefinition.revision')
                        echo "The revision number is $LATEST_REVISION"
                        aws ecs update-service \
                            --cluster LearnJenkinsApp-Cluster-Prod \
                            --service LearnJenkinsApp-Service-Prod \
                            --task-definition LearnJenkinsApp-TaskDef-Prod:$LATEST_REVISION
                    '''
                }
            }
        }
    }
}
