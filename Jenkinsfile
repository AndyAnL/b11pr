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
                    // Количество попыток
                    int retries = 5
                    // Задержка между попытками (в секундах)
                    int delay = 5

                    // Цикл для повторного запроса
                    for (int i = 0; i < retries; i++) {
                        try {
                            // Проверка кода ответа
                            def responseCode = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${HOST_PORT}/${HTML_FILE}", returnStdout: true).trim()
                            
                            // Если код ответа 200, завершаем цикл
                            if (responseCode == '200') {
                                echo "HTTP response code is 200. Nginx is ready."
                                break
                            } else {
                                echo "Attempt ${i + 1}: HTTP response code is ${responseCode}. Retrying in ${delay} seconds..."
                            }
                        } catch (Exception e) {
                            echo "Attempt ${i + 1}: Failed to connect to Nginx. Retrying in ${delay} seconds..."
                        }

                        // Задержка перед следующей попыткой
                        sleep delay
                    }

                    // Если после всех попыток код ответа не 200, завершаем с ошибкой
                    def finalResponseCode = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${HOST_PORT}/${HTML_FILE}", returnStdout: true).trim()
                    if (finalResponseCode != '200') {
                        error "HTTP response code is not 200 after ${retries} attempts. Got ${finalResponseCode}"
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
            steps