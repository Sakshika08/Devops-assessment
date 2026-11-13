pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'test -f Dockerfile'
                sh "docker build -t sakshika/my-assessment-app:${BUILD_NUMBER} ."
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'U', passwordVariable: 'P')]) {
                    sh "echo \$P | docker login -u \$U --password-stdin"
                    sh "docker push sakshika/my-assessment-app:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' k8s/deployment.yaml"
                sh "kubectl apply -f k8s/deployment.yaml"
            }
        }
    }
}
