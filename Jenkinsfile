pipeline {
    agent any

    environment {
        IMAGE_NAME = "4ddocker/lab1:${env.BUILD_NUMBER}"
        IMAGE_LATEST = "4ddocker/lab1:latest"
        LOCAL_DATA_PATH = "C:\\DopEdu\\ML_ITMO\\DevOpsLab\\Lab1"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📦 Клонирование репозитория из GitHub...'
                checkout scm
                echo '✅ Код успешно получен'
            }
        }

        stage('Copy Large Files') {
            steps {
                echo '📁 Копирование больших файлов (данные и модель) из локальной папки...'
                bat """
                    if not exist "data" mkdir data
                    xcopy /E /I /Y "${LOCAL_DATA_PATH}\\data\\*" "data\\"
                    if not exist "models" mkdir models
                    xcopy /E /I /Y "${LOCAL_DATA_PATH}\\models\\*" "models\\"
                """
                echo '✅ Большие файлы скопированы'
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
                    docker run -d --name test-container-${BUILD_NUMBER} -p 8888:8000 ${IMAGE_NAME}
                    timeout /t 10 /nobreak > nul
                    curl -f http://localhost:8888/health || exit 1
                    docker stop test-container-${BUILD_NUMBER}
                    docker rm test-container-${BUILD_NUMBER}
                """
                echo '✅ Контейнер успешно протестирован'
            }
        }
    }

    post {
        always {
            script {
                bat "docker rmi ${IMAGE_NAME} ${IMAGE_LATEST} || true"
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