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
                    // Build Docker image
                    withDockerServer([uri: "tcp://localhost:2375"]) {
                        script {
                            sh "docker build -t my-nginx-wordpress-image -f /root/Dockerfile "
                        }
                    }
                    // Push Docker image to registry
                    docker.withRegistry('https://hub.docker.com/', '785a3777-2313-4836-81fb-3f8e1f596082') {
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

                    // Apply Kubernetes manifests
                    sh "kubectl apply -f kubernetes/nginx-deployment.yaml -n ${kubernetesNamespace}"
                    sh "kubectl apply -f kubernetes/nginx-service.yaml -n ${kubernetesNamespace}"
                }
            }
        }
    }
}
