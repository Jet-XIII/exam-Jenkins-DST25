pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds-jet' 
        DOCKERHUB_USER = 'jet13'
        REGISTRY = "docker.io"
        KUBECONFIG = credentials('config') // Récupère le kubeconfig depuis les credentials Jenkins
    }

    stages {

        stage('Checkout') {
            steps {
                deleteDir() // Nettoie le workspace
                git url: 'https://github.com/Jet-XIII/exam-Jenkins-DST25.git', branch: "${env.BRANCH_NAME}"
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Cast Service') {
                    steps {
                        script {
                            docker.build("${DOCKERHUB_USER}/cast-service", "./cast-service")
                        }
                    }
                }
                stage('Build Movie Service') {
                    steps {
                        script {
                            docker.build("${DOCKERHUB_USER}/movie-service", "./movie-service")
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    script {
                        sh '''
                            echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        '''
                        docker.image("${DOCKERHUB_USER}/cast-service").push("latest")
                        docker.image("${DOCKERHUB_USER}/movie-service").push("latest")
                    }
                }
            }
        }

        stage('Test K8s Access') {
            steps {
                sh 'kubectl get ns'
            }
        }

        stage('Helm Deploy to Dev') {
            when {
                branch 'dev'
            }
            steps {
                sh 'helm upgrade --install cast-service ./charts --namespace dev --set image.repository=docker.io/${DOCKERHUB_USER}/cast-service,image.tag=latest'
                sh 'helm upgrade --install movie-service ./charts --namespace dev --set image.repository=docker.io/${DOCKERHUB_USER}/movie-service,image.tag=latest'
            }
        }

        stage('Helm Deploy to QA') {
            when {
                branch 'qa'
            }
            steps {
                sh 'helm upgrade --install cast-service ./charts --namespace qa --set image.repository=docker.io/${DOCKERHUB_USER}/cast-service,image.tag=latest'
                sh 'helm upgrade --install movie-service ./charts --namespace qa --set image.repository=docker.io/${DOCKERHUB_USER}/movie-service,image.tag=latest'
            }
        }

        stage('Helm Deploy to Staging') {
            when {
                branch 'staging'
            }
            steps {
                sh 'helm upgrade --install cast-service ./charts --namespace staging --set image.repository=docker.io/${DOCKERHUB_USER}/cast-service,image.tag=latest'
                sh 'helm upgrade --install movie-service ./charts --namespace staging --set image.repository=docker.io/${DOCKERHUB_USER}/movie-service,image.tag=latest'
            }
        }

        stage('Manual Approval for Production') {
            when {
                branch 'master'
            }
            steps {
                input message: "Déployer en production ?", ok: "Oui, déployer"
                sh 'helm upgrade --install cast-service ./charts --namespace prod --set image.repository=docker.io/${DOCKERHUB_USER}/cast-service,image.tag=latest'
                sh 'helm upgrade --install movie-service ./charts --namespace prod --set image.repository=docker.io/${DOCKERHUB_USER}/movie-service,image.tag=latest'
            }
        }
    }
}
