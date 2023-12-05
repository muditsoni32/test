pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/muditsoni32/test.git'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def imageName = 'nginx-app'
                    def dockerImage = docker.build(imageName)
                    docker.withRegistry('https://hub.docker.com', '785a3777-2313-4836-81fb-3f8e1f596082') {
                        dockerImage.push('latest')
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
