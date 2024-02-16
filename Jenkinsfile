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
                    // Build Docker image1
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
                    def dataSecretName = 'data-secret' // Name of the Kubernetes secret
                    def dataSecretKey = 'data.txt' // Key of the data file in the secret

                    withCredentials([file(credentialsId: kubeconfigCredentialId, variable: 'KUBECONFIG')]) {

                        // Create the Kubernetes secret if it doesn't exist
                        sh "kubectl create secret generic ${dataSecretName} --from-file=data.txt || true"

                        sh """
                        export KUBECONFIG=\$KUBECONFIG
                        sudo kubectl apply -f storage-class.yaml -n ${kubernetesNamespace}
                        sudo kubectl apply -f pv-pvc.yaml -n ${kubernetesNamespace}
                        sudo kubectl apply -f nginx-deployment.yaml -n ${kubernetesNamespace}
                        sudo kubectl apply -f nginx-service.yaml -n ${kubernetesNamespace}
                        sudo kubectl apply -f nginx-config.yaml -n ${kubernetesNamespace}
                        """

                        // Mount the Kubernetes secret as a volume in the deployment
                        sh """
                        export KUBECONFIG=\$KUBECONFIG
                        sudo kubectl patch deployment nginx-deployment -n ${kubernetesNamespace} --patch "
                        {
                          'spec': {
                            'template': {
                              'spec': {
                                'volumes': [
                                  {
                                    'name': 'data-volume',
                                    'secret': {
                                      'secretName': '${dataSecretName}',
                                      'items': [
                                        {
                                          'key': '${dataSecretKey}',
                                          'path': 'data.txt'
                                        }
                                      ]
                                    }
                                  }
                                ],
                                'containers': [
                                  {
                                    'name': 'nginx',
                                    'volumeMounts': [
                                      {
                                        'name': 'data-volume',
                                        'mountPath': '/usr/share/nginx/'
                                      }
                                    ]
                                  }
                                ]
                              }
                            }
                          }
                        }
                        "
                        """
                    }
                }
            }
        }
    }
} // This is the missing closing curly brace
