pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds-jet'
        DOCKERHUB_USER = 'jet13'
        REGISTRY = "docker.io"
        KUBECONFIG = credentials('config')
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

        stage('Helm Deploy to Dev') {
            when { branch 'dev' }
            steps {
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace dev fastapiapp-db-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
                    helm upgrade --install fastapiapp-dev ./fastapiapp --namespace dev \
                        --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                        --set cast.image.tag=latest \
                        --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                        --set movie.image.tag=latest \
                        --set global.env=dev \
                        --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }

        stage('Helm Deploy to QA') {
            when { branch 'qa' }
            steps {
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace qa fastapiapp-db-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
                    helm upgrade --install fastapiapp-qa ./fastapiapp --namespace qa \
                        --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                        --set cast.image.tag=latest \
                        --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                        --set movie.image.tag=latest \
                        --set global.env=qa \
                        --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }

        stage('Helm Deploy to Staging') {
            when { branch 'staging' }
            steps {
                sh '''
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace staging fastapiapp-db-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
                    helm upgrade --install fastapiapp-staging ./fastapiapp --namespace staging \
                        --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                        --set cast.image.tag=latest \
                        --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                        --set movie.image.tag=latest \
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
                    export POSTGRES_PASSWORD=$(kubectl get secret --namespace master fastapiapp-db-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
                    helm upgrade --install fastapiapp-master ./fastapiapp --namespace master \
                        --set cast.image.repository=docker.io/${DOCKERHUB_USER}/cast-service \
                        --set cast.image.tag=latest \
                        --set movie.image.repository=docker.io/${DOCKERHUB_USER}/movie-service \
                        --set movie.image.tag=latest \
                        --set global.env=master \
                        --set global.postgresql.auth.postgresPassword=$POSTGRES_PASSWORD
                '''
            }
        }
    }
}
