pipeline {
    agent any

    environment {
        IMAGE_NAME = "4ddocker/lab1:${env.BUILD_NUMBER}"
        IMAGE_LATEST = "4ddocker/lab1:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📦 Клонирование репозитория из GitHub...'
                checkout scm
                echo '✅ Код успешно получен'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🏗️ Сборка Docker образа...'
                bat "docker build -t ${IMAGE_NAME} ."
                bat "docker tag ${IMAGE_NAME} ${IMAGE_LATEST}"
                echo '✅ Образ собран'
            }
        }

        stage('Test Container') {
            steps {
                echo '🧪 Запуск тестового контейнера...'
                bat """
                    docker run -d --name test-container-${env.BUILD_NUMBER} -p 8888:8000 ${IMAGE_NAME}
                    timeout /t 10 /nobreak > nul
                    curl -f http://localhost:8888/health
                    docker stop test-container-${env.BUILD_NUMBER}
                    docker rm test-container-${env.BUILD_NUMBER}
                """
                echo '✅ Контейнер успешно протестирован'
            }
        }
    }

    post {
        always {
            script {
                // Используем прямые имена, а не переменные в bat
                bat "docker rmi 4ddocker/lab1:${env.BUILD_NUMBER} 4ddocker/lab1:latest || true"
            }
        }
        success {
            echo '🎉 Pipeline успешно выполнен!'
        }
        failure {
            echo '❌ Pipeline завершился с ошибкой. Проверьте логи выше.'
        }
    }
}