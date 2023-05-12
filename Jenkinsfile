pipeline {
    agent any

    environment {
        CHAT_TOKEN = credentials('google-chat-guisousa')
    }

    stages {
        stage('Atualizando Código') {
            steps {
                git branch: 'master', url: 'https://github.com/gui-sousa/bwg-server-update.git'
            }

            post {
                always {
                    hangoutsNotify message: "⚙️ Iniciando build\n⏰ Horario de início: $BUILD_TIMESTAMP", token: "$CHAT_TOKEN", threadByJob: false
                }
            }
        }

        stage('Deploy') {
            steps {
                sh 'ansible-playbook -i hosts.ini playbook.yaml -vvv'
            }
        }
    }

    post {
        success {
            hangoutsNotify {
                message: "✅ Deu Certo!\n⏰ Tempo de Duração: ${currentBuild.duration / 1000} segundos",
                token: "$CHAT_TOKEN",
                threadByJob: false
            }

            emailext {
                to: "gsousa@bwg.com.br",
                subject: "Build Notification: ${JOB_NAME}-Build# ${BUILD_NUMBER} ${currentBuild.result}",
                from: "jenkins@bwg.com.br",
                body: "${currentBuild.result}: ${BUILD_URL}",
                attachLog: true,
                compressLog: true
            }
        }

        failure {
            hangoutsNotify {
                message: "❌ Deu Errado!\n⏰ Tempo de Duração: ${currentBuild.duration / 1000} segundos",
                token: "$CHAT_TOKEN",
                threadByJob: false
            }

            emailext {
                to: "gsousa@bwg.com.br",
                subject: "Build Notification: ${JOB_NAME}-Build# ${BUILD_NUMBER} ${currentBuild.result}",
                from: "jenkins@bwg.com.br",
                body: "${currentBuild.result}: ${BUILD_URL}",
                attachLog: true,
                compressLog: true
            }
        }
    }
}
