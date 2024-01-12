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
                    sh "cd /home/jenkins && docker build -t my-nginx-wordpress-image ."
                    sh "sudo docker image tag my-nginx-wordpress-image:latest muditsoni32/my-nginx-wordpress-image:latest"

                    // Push Docker image to registry
                    docker.withRegistry('https://registry.hub.docker.com', '785a3777-2313-4836-81fb-3f8e1f596082') {
                        docker.image('muditsoni32/my-nginx-wordpress-image:latest').push()
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
                            sudo kubectl apply -f storage-class.yaml -n ${kubernetesNamespace}
                            sudo kubectl apply -f pv-pvc.yaml -n ${kubernetesNamespace}
                            sudo kubectl apply -f nginx-deployment.yaml -n ${kubernetesNamespace}
                            sudo kubectl apply -f nginx-service.yaml -n ${kubernetesNamespace}
                            sudo kubectl apply -f nginx-config.yaml -n ${kubernetesNamespace}
                        """
                    }
                }
            }
        }

        stage('Transfer Code to Kubernetes Pod') {
            steps {
                script {
                    def podName = 'github-transfer-pod'

                    // Apply Kubernetes YAML with initContainer for GitHub clone
                    sh """
                        kubectl apply -f github-transfer-pod.yaml
                        kubectl wait --for=condition=ready pod/${podName} --timeout=300s
                        kubectl logs ${podName}
                    """
                }
            }
        }
    }

    post {
        always {
            // Cleanup
            script {
                sh "kubectl delete pod github-transfer-pod"
            }
        }
    }
}
