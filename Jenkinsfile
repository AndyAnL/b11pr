pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AndyAnL/b11pr.git'
            }
        }

        stage('Run Nginx Container') {
            steps {
                script {
                    sh """
                        docker run -d --name nginx-container -p 9889:80 \
                        -v \$(pwd)/index.html:/usr/share/nginx/html/index.html \
                        nginx
                    """
                }
            }
        }

        stage('Check HTTP Response') {
            steps {
                script {
                    def responseCode = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:9889/index.html", returnStdout: true).trim()
                    if (responseCode != '200') {
                        error "HTTP response code is not 200. Got ${responseCode}"
                    }
                }
            }
        }

        stage('Compare MD5 Sums') {
            steps {
                script {
                    def localMD5 = sh(script: "md5sum index.html | awk '{print \$1}'", returnStdout: true).trim()
                    def containerMD5 = sh(script: "curl -s http://localhost:9889/index.html | md5sum | awk '{print \$1}'", returnStdout: true).trim()
                    if (localMD5 != containerMD5) {
                        error "MD5 sums do not match. Local: ${localMD5}, Container: ${containerMD5}"
                    }
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: 'Jenkins Build Success',
                body: 'The Jenkins build was successful!',
                to: 'andyan@bk.ru'
            )
        }
        failure {
            emailext (
                subject: 'Jenkins Build Failed',
                body: 'The Jenkins build has failed. Please check the logs.',
                to: 'andyan@bk.ru'
            )
        }
    }
}