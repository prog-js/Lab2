pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = '4ddocker'
        DOCKER_HUB_PASS = credentials('docker')
        IMAGE_NAME = "${DOCKER_HUB_USER}/lab1:${env.BUILD_NUMBER}"
        IMAGE_LATEST = "${DOCKER_HUB_USER}/lab1:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📦 Клонирование репозитория из GitHub...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/prog-js/DevOPS_Lab1.git',
                        credentialsId: 'github-token'
                    ]]
                ])
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
                 echo '🧪 Запуск тестового контейнера с монтированием данных...'
                 bat """
            docker run -d --name test-container-${BUILD_NUMBER} \
                -p 8888:8000 \
                -v "C:\\DopEdu\\ML_ITMO\\DevOps_BigFiles\\data:/data" \
                -v "C:\\DopEdu\\ML_ITMO\\DevOps_BigFiles\\models:/models" \
                4ddocker/lab1:${BUILD_NUMBER}
            
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
                bat 'docker logout || true'
            }
        }
        success {
            echo '🎉 CI/CD Pipeline успешно выполнен!'
        }
        failure {
            echo '❌ Pipeline завершился с ошибкой. Проверьте логи выше.'
        }
    }
}