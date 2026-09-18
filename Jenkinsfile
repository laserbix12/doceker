pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
        timeout(time: 15, unit: 'MINUTES')
    }

    environment {
        DOCKER_IMAGE = 'michaell_pulido/doceker'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        CI_VENV = "/tmp/jenkins-venv-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Obteniendo el código configurado en Jenkins...'
                checkout scm
            }
        }

        stage('Testing') {
            steps {
                echo 'Instalando dependencias y ejecutando validaciones Django...'
                sh '''
                    set -eu
                    python3 --version
                    python3 -m venv "${CI_VENV}"
                    "${CI_VENV}/bin/python" -m pip install --upgrade pip
                    "${CI_VENV}/bin/python" -m pip install --no-cache-dir -r requirements.txt
                    "${CI_VENV}/bin/python" manage.py check
                    "${CI_VENV}/bin/python" manage.py test
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
            sh 'rm -rf "${CI_VENV}"'
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
