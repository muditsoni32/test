pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git credentialsId: 'af4756ab-cbbe-41a1-ad9d-7ca7b664cabd', url: 'https://github.com/muditsoni32/test.git'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.build('muditsoni32/my-nginx-wordpress-image:b1')
                    docker.withRegistry('https://docker-registry.example.com', 'credentials-id') {
                        docker.image('muditsoni32/my-nginx-wordpress-image:b1').push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def kubernetesNamespace = 'default'
                    def kubernetesServiceName = 'nginx-service'

                    sh "kubectl apply -f kubernetes/nginx-deployment.yaml -n ${kubernetesNamespace}"
                    sh "kubectl apply -f kubernetes/nginx-service.yaml -n ${kubernetesNamespace}"
                }
            }
        }
    }
}
