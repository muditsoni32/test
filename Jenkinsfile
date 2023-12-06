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
                        
                         sh "cd /home/jenkins && docker build -t nginx-app ."

                    
                    // Push Docker image to registry
                    docker.withRegistry('https://registry.hub.docker.com', '785a3777-2313-4836-81fb-3f8e1f596082') {
                        docker.image('muditsoni32/nginx-app:latest').push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def kubernetesNamespace = 'default'
                    def kubeconfigCredentialId = '1c90b3a7-fd8c-44a2-95e4-a093fc950bea'
                    
                    withCredentials([file(credentialsId: kubeconfigCredentialId, variable: 'KUBECONFIG')]) {

                     sh """
                    export KUBECONFIG=\$KUBECONFIG
                    sudo kubectl apply -f nginx-deployment.yaml -n ${kubernetesNamespace}
                    sudo kubectl apply -f nginx-service.yaml -n ${kubernetesNamespace}
                """
                }
            }
        }
    }
}
}