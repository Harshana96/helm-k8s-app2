pipeline {
    agent any

    environment {
        HETZNER_HOST = '89.167.27.46'
        HETZNER_USER = 'root'
        REPO_URL     = 'https://github.com/Harshana96/helm-k8s-app2.git'
        APP_DIR      = 'helm-k8s-app2'
    }

    stages {

        stage('Clone Repo on Hetzner') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${HETZNER_USER}@${HETZNER_HOST} '
                            if [ -d ~/${APP_DIR} ]; then
                                cd ~/${APP_DIR} && git pull
                            else
                                git clone ${REPO_URL} ~/${APP_DIR}
                            fi
                        '
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${HETZNER_USER}@${HETZNER_HOST} '
                            cd ~/${APP_DIR}
                            export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                            kubectl create namespace dev --dry-run=client -o yaml | kubectl apply -f -
                            helm upgrade --install nginx-app2-dev ./helm \
                                -f ./helm/values-dev.yaml \
                                --namespace dev \
                                --atomic \
                                --timeout 60s
                        '
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'hetzner-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${HETZNER_USER}@${HETZNER_HOST} '
                            export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
                            kubectl get pods -n dev
                            kubectl get ingress -n dev
                        '
                    """
                }
            }
        }
    }

    post {
        success { echo '✅ App2 deployed to Dev successfully!' }
        failure { echo '❌ App2 deployment failed!' }
    }
}
