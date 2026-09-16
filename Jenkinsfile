pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.11'
        IMAGE_NAME = 'api_test'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Clonando repositorio...'
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                echo 'Instalando dependencias del proyecto...'
                sh '''
                    python --version
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run checks') {
            steps {
                echo 'Validando la aplicación Django...'
                sh 'python manage.py check'
            }
        }

        stage('Run tests') {
            steps {
                echo 'Ejecutando pruebas...'
                sh 'python manage.py test'
            }
        }

        stage('Build Docker image') {
            steps {
                echo 'Construyendo imagen Docker...'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Desplegando en Kubernetes...'
                sh '''
                    kubectl apply -f k8s/postgres-configmap.yaml
                    kubectl apply -f k8s/postgres-secret.yaml
                    kubectl apply -f k8s/backend-deployment.yaml
                    kubectl apply -f k8s/backend-service.yaml
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finalizado.'
        }
        success {
            echo 'La pipeline se ejecutó correctamente.'
        }
        failure {
            echo 'La pipeline falló.'
        }
    }
}
