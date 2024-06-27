pipeline {
    agent any

    environment {
        KUBECONFIG = credentials('kubernetes-config')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/aaraispvtltd/mosquitto_kubernetes.git']]])
            }
        }

        stage('Create Secret') {
            steps {
                script {
                    // Create TLS Secret
                    sh 'kubectl create secret tls mosquitto-tls-secret --cert=${WORKSPACE}/kubernetes/certs/server.crt --key=${WORKSPACE}/kubernetes/certs/server.key --dry-run=client -o yaml | kubectl apply -f -'
                    // Create CA Cert Secret (if needed)
                    sh 'kubectl create secret generic mosquitto-cacert --from-file=ca.crt=${WORKSPACE}/kubernetes/certs/ca.crt --dry-run=client -o yaml | kubectl apply -f -'
                }
            }
        }

        stage('Deploy Mosquitto') {
            steps {
                script {
                    // Apply Kubernetes resources
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-configmap.yaml'
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-deployment.yaml'
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-service.yaml'
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-ingress.yaml'
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-pvc.yaml'
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-storage.yaml'
                    sh 'kubectl apply -f ${WORKSPACE}/kubernetes/kubernetes/mosquitto-hpa.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}
