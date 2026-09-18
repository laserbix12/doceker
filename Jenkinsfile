pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 15, unit: 'MINUTES')
    }

    environment {
        // El repositorio Docker coincide con el remoto Git configurado en este proyecto.
        DOCKER_IMAGE = 'laserbix12/doceker'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Obteniendo el código de la rama main...'
                git branch: 'main', url: 'https://github.com/laserbix12/doceker.git'
            }
        }

        stage('Testing') {
            agent {
                docker {
                    image 'python:3.11-slim'
                    reuseNode true
                }
            }
            steps {
                echo 'Instalando dependencias y ejecutando validaciones Django...'
                sh '''
                    set -eu
                    python --version
                    python -m pip install --upgrade pip
                    python -m pip install --no-cache-dir -r requirements.txt
                    python manage.py check
                    python manage.py test
                '''
            }
        }

        stage('Build') {
            steps {
                echo "Construyendo ${DOCKER_IMAGE}:${IMAGE_TAG}..."
                sh '''
                    set -eu
                    docker build \
                        --tag "${DOCKER_IMAGE}:${IMAGE_TAG}" \
                        --tag "${DOCKER_IMAGE}:latest" \
                        .
                '''
            }
        }

        stage('Push') {
            steps {
                echo 'Publicando la imagen en Docker Hub...'
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set -eu
                        trap 'docker logout >/dev/null 2>&1 || true' EXIT
                        printf '%s' "${DOCKER_PASSWORD}" | docker login \
                            --username "${DOCKER_USERNAME}" \
                            --password-stdin
                        docker push "${DOCKER_IMAGE}:${IMAGE_TAG}"
                        docker push "${DOCKER_IMAGE}:latest"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizado.'
        }
        success {
            echo "Imagen publicada: ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }
        failure {
            echo 'La pipeline falló. Revisa los logs de la etapa indicada.'
        }
    }
}
