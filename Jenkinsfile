pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "aakash11m/machines-efficiency-prediction"
        DOCKER_HUB_CREDENTIALS_ID = "gitops-dockerhub-token"
    }
    stages {
        stage('Checkout Github') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'gitHub-token',
                        url: 'https://github.com/Aakash5937k/machines-efficiency-prediction.git'
                    ]]
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    dockerImage = docker.build("${DOCKER_HUB_REPO}:latest")
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                script {
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKER_HUB_CREDENTIALS_ID}") {
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Install Kubectl & ArgoCD CLI') {
            steps {
                sh '''
                    echo 'Installing Kubectl & ArgoCD CLI...'

                    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

                    chmod +x kubectl
                    sudo mv kubectl /usr/local/bin/kubectl

                    sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64

                    sudo chmod +x /usr/local/bin/argocd
                '''
            }
        }

       stage('Apply Kubernetes & Sync App with ArgoCD') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        echo 'Checking Kubernetes connection...'
                        kubectl get nodes

                        echo 'Logging in to ArgoCD...'
                        argocd login 34.59.198.187:30245 \
                            --username admin \
                            --password "$(kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d)" \
                            --insecure

                        echo 'Syncing ArgoCD application...'
                        argocd app sync gitopsapp
                    '''
                }
            }
        }
    }
}

