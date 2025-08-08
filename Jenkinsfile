pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds-jet'
        DOCKERHUB_USER = 'jet13'
        REGISTRY = "docker.io"
        KUBECONFIG = credentials('config') // kubeconfig est injecté comme fichier de credentials
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
                sh '''
                    echo ">>> Test Kube Access"
                    kubectl config current-context
                    kubectl get ns
                    env | grep KUBECONFIG || true
                '''
            }
        }

        stage('Helm Deploy to Dev') {
            when { branch 'dev' }
            steps {
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace dev fastapiapp-dev-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

                    helm upgrade --install fastapiapp-dev ./fastapiapp \
                      --namespace dev \
                      --create-namespace \
                      --set castService.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                      --set castService.image.tag=latest \
                      --set movieService.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                      --set movieService.image.tag=latest \
                      --set global.env=dev \
                      --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }

        stage('Helm Deploy to QA') {
            when { branch 'qa' }
            steps {
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace qa fastapiapp-qa-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

                    helm upgrade --install fastapiapp-qa ./fastapiapp \
                      --namespace qa \
                      --create-namespace \
                      --set castService.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                      --set castService.image.tag=latest \
                      --set movieService.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                      --set movieService.image.tag=latest \
                      --set global.env=qa \
                      --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }

        stage('Helm Deploy to Staging') {
            when { branch 'staging' }
            steps {
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace staging fastapiapp-staging-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

                    helm upgrade --install fastapiapp-staging ./fastapiapp \
                      --namespace staging \
                      --create-namespace \
                      --set castService.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                      --set castService.image.tag=latest \
                      --set movieService.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                      --set movieService.image.tag=latest \
                      --set global.env=staging \
                      --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }

        stage('Manual Approval for Production') {
            when { branch 'master' }
            steps {
                input message: "Déployer en production ?", ok: "Oui, déployer"
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace prod fastapiapp-prod-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

                    helm upgrade --install fastapiapp-prod ./fastapiapp \
                      --namespace prod \
                      --create-namespace \
                      --set castService.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                      --set castService.image.tag=latest \
                      --set movieService.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                      --set movieService.image.tag=latest \
                      --set global.env=prod \
                      --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }
    }
}
