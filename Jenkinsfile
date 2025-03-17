pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/AndyAnL/b11pr.git' // Укажите ваш репозиторий
        CONTAINER_NAME = 'nginx-container'
        HOST_PORT = '9889'
        HTML_FILE = 'index.html'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Check Changes') {
            steps {
                script {
                    // Получаем список измененных файлов
                    def changes = sh(script: "git diff --name-only HEAD~1 HEAD", returnStdout: true).trim()
                    
                    // Проверяем, изменен ли index.html
                    if (!changes.contains('index.html')) {
                        echo 'No changes in index.html. Skipping build.'
                        currentBuild.result = 'SUCCESS' // Пропустить сборку
                        return
                    }
                }
            }
        }

        stage('Run Nginx Container') {
            steps {
                script {
                    // Запуск контейнера с nginx
                    sh """
                        docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:80 \
                        -v \$(pwd)/${HTML_FILE}:/usr/share/nginx/html/${HTML_FILE} \
                        nginx
                    """
                }
            }
        }

        stage('Check HTTP Response') {
            steps {
                script {
                    // Проверка кода ответа
                    def responseCode = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${HOST_PORT}/${HTML_FILE}", returnStdout: true).trim()
                    if (responseCode != '200') {
                        error "HTTP response code is not 200. Got ${responseCode}"
                    }
                }
            }
        }

        stage('Compare MD5 Sums') {
            steps {
                script {
                    // Вычисление MD5 локального файла
                    def localMD5 = sh(script: "md5sum ${HTML_FILE} | awk '{print \$1}'", returnStdout: true).trim()

                    // Вычисление MD5 файла из контейнера
                    def containerMD5 = sh(script: "curl -s http://localhost:${HOST_PORT}/${HTML_FILE} | md5sum | awk '{print \$1}'", returnStdout: true).trim()

                    // Сравнение MD5
                    if (localMD5 != containerMD5) {
                        error "MD5 sums do not match. Local: ${localMD5}, Container: ${containerMD5}"
                    }
                }
            }
        }

        stage('Send Email Notification') {
            steps {
                script {
                    // Отправка email в случае успеха
                    emailext (
                        subject: 'Jenkins CI: Build Successful',
                        body: 'The Jenkins CI pipeline has completed successfully.',
                        to: 'andyan@bk.ru' // Укажите ваш email
                    )
                }
            }
        }
    }

    post {
        always {
            script {
                // Остановка и удаление контейнера
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
            }
        }
        failure {
            script {
                // Отправка email в случае ошибки
                emailext (
                    subject: 'Jenkins CI: Build Failed',
                    body: 'The Jenkins CI pipeline has failed. Please check the logs.',
                    to: 'andyan@bk.ru' // Укажите ваш email
                )
            }
        }
    }
}