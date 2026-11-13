pipeline {
    agent any

    environment {
        AWS_REGION   = 'ap-south-1'
        ECR_REGISTRY = '://amazonaws.com'
        IMAGE_NAME   = 'my-assessment-app'
    }

    stages {
        stage('Test') {
            steps {
                sh 'test -f Dockerfile'
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${ECR_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        docker push ${ECR_REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        kubectl delete secret ecr-secret --ignore-not-found
                        kubectl create secret docker-registry ecr-secret \
                          --docker-server=${ECR_REGISTRY} \
                          --docker-username=AWS \
                          --docker-password=\$(aws ecr get-login-password --region ${AWS_REGION})
                        sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' k8s/deployment.yaml
                        kubectl apply -f k8s/deployment.yaml
                    """
                }
            }
        }
    }
}
