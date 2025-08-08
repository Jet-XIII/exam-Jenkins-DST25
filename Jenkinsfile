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
                deleteDir()
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

        // 🔽 NOUVEAU STAGE: déploiement PostgreSQL via Helm
        stage('Helm Deploy Database') {
            when { branch 'dev' }
            steps {
                sh '''
                    helm dependency update fastapiapp
                    helm upgrade --install fastapiapp-db fastapiapp \
                    --namespace dev --create-namespace
                '''
            }
        }

        stage('Helm Deploy to Dev') {
            when { branch 'dev' }
            steps {
                sh '''
                    helm upgrade --install fastapiapp-dev ./fastapiapp --namespace dev \
                    --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                    --set cast.image.tag=latest \
                    --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                    --set movie.image.tag=latest
                '''
            }
        }

        stage('Helm Deploy to QA') {
            when { branch 'qa' }
            steps {
                sh '''
                    helm upgrade --install fastapiapp-qa ./fastapiapp --namespace qa \
                    --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                    --set cast.image.tag=latest \
                    --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                    --set movie.image.tag=latest
                '''
            }
        }

        stage('Helm Deploy to Staging') {
            when { branch 'staging' }
            steps {
                sh '''
                    helm upgrade --install fastapiapp-staging ./fastapiapp --namespace staging \
                    --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                    --set cast.image.tag=latest \
                    --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                    --set movie.image.tag=latest
                '''
            }
        }

        stage('Manual Approval for Production') {
            when { branch 'master' }
            steps {
                input message: "Déployer en production ?", ok: "Oui, déployer"
                sh '''
                    helm upgrade --install fastapiapp-prod ./fastapiapp --namespace prod \
                    --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                    --set cast.image.tag=latest \
                    --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                    --set movie.image.tag=latest
                '''
            }
        }

        /*
        stage('Healthcheck (Dev)') {
            when { branch 'dev' }
            steps {
                sh '''
                    sleep 10
                    kubectl run curlpod --rm -i --restart=Never --image=curlimages/curl:latest -n dev -- \
                    curl http://fastapiapp-cast.dev.svc.cluster.local:8000/health
                '''
            }
        }
        */
    }
}
