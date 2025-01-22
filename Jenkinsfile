pipeline {
    agent any   //Пайплайн выполняется на локальных и удалённых машинах

    environment {
        IMAGE_NAME = 'avs'   //Имя докер-контейнера, -образа.
        DOCKER_IMAGE = 'avs:latest'
        PORT = '8085' // Порт для использования контейнером
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/xCredo/avs.git'  // Клонируем ветку с репозитория
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."  //Собираем докер-образ
                }
            }
        }

        stage('Deploy Docker Container') {
            steps {
                script {
                    // Проверяем запущен ли контейнер, описанный в переиенных окружения
                    // и если запущен,тогда останавливаем и удаляем 
                    sh """
                        if [ \$(docker ps -aq -f name=${IMAGE_NAME}) ]; then
                            docker stop ${IMAGE_NAME}
                            docker rm ${IMAGE_NAME}
                        fi
                    """
                    //// Проверяем занят ли порт и еслион занят,
                    //// тогда заершаем процессф на порту
                    sh """
                        if ss -tuln | grep :${PORT}; then
                            echo "Port ${PORT} is in use. Freeing it..."
                            fuser -k ${PORT}/tcp || true
                        fi
                    """
                    // Запускаем новый контейнер, пробрасывает порт 8181 на порт 80 внутри контейнера
                    // и указыавем имя образа
                    sh "docker run -d --name ${IMAGE_NAME} -p 8181:80 ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {  // Описание вывдимых сообщений в случае успешного/неуспешного выполнения пайплайна.
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo 'Deployment failed. Check logs for details.'
        }
    }
}
