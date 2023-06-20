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
    
    stage('Atualizando Servidores de Banco de Dados') {
      steps {
        sh 'ansible-playbook -i winUpdate-hosts.ini bd-playbook.yaml -vvv'
        sh 'ansible-playbook -i winUpdate-hosts.ini bd-playbook.yaml -vvv'
      }
    }

    stage('Aprovação') {
        steps {
          input "Deseja continuar para os servidores de Aplicação?"
        }

        post {
            always {
                hangoutsNotify message: "✅ Atualização dos Servidores de Banco de Dados Concluida!\n🤖 Aguarando Aprovação para Prosseguimento\n⏰ Tempo decorrido: $BUILD_TIMESTAMP", token: "$CHAT_TOKEN", threadByJob: false
            }
        }
    }

    stage('Atualizando Servidores de Aplicação') {
      steps {
        sh 'ansible-playbook -i winUpdate-hosts.ini prod-playbook.yaml -vvv'
        sh 'ansible-playbook -i winUpdate-hosts.ini prod-playbook.yaml -vvv'
      }
    }

    stage('Iniciando serviços ApServers') {
      steps {
        sh 'ansible-playbook -i /restart-winservices/winUpdate-hosts.ini /restart-winservices/apServer-playbook.yaml -vvv'
      }
    }

    stage('Aguardando inicialização ApServers') {
      steps {
        script {
            sleep(time: 600, unit: 'SECONDS')
        }
      }
    }

    stage('Iniciando demais serviços') {
      steps {
        sh 'ansible-playbook -i /restart-winservices/winUpdate-hosts.ini /restart-winservices/apIntegration-playbook.yaml --extra-vars "service_state=start" -vvv'
        sh 'ansible-playbook -i /restart-winservices/winUpdate-hosts.ini /restart-winservices/apIntegration-playbook.yaml --extra-vars "service_state=restart" -vvv'
      }
    }

    

    stage('Testando Portais') {
      steps {
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "https://meuportalrh.com.br/site/.net/index.ashx/GetPublicLinks", validResponseCodes: '200', validResponseContent: '"success":true'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "https://messer.meuportalrh.com.br/site/.net/index.ashx/GetPublicLinks", validResponseCodes: '200', validResponseContent: '"success":true'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "https://folha.4bee.com.br/4beeFopag/servlet/generic.LoginServlet#/home", validResponseCodes: '200', validResponseContent: 'Entrar'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "https://folha.novacoop.com.br/4beeFopag/servlet/generic.LoginServlet#/home", validResponseCodes: '200', validResponseContent: 'OlÃ¡, sÃ³cio cooperado!'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "https://meuportalrh.com.br/site/", validResponseCodes: '200', validResponseContent: 'main.html'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "http://10.1.85.10:17881/wsdl/IApWebServices", validResponseCodes: '200', validResponseContent: 'This XML file'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "http://10.1.85.6:9050/wsdl/IApWebServices", validResponseCodes: '200', validResponseContent: 'This XML file'
        httpRequest consoleLogResponseBody: true, responseHandle: 'NONE', url: "http://10.1.85.6:8060/soap/IApWebServices", validResponseCodes: '200', validResponseContent: 'WSDL'
      }  
    }

  }

  post {
      success {
           hangoutsNotify message: "✅ Deu Certo!\n⏰ Tempo de Duração: ${currentBuild.duration / 1000} segundos", token: "$CHAT_TOKEN", threadByJob: false
           emailext to: "gsousa@bwg.com.br",
           subject: "Build Notification: ${JOB_NAME}-Build# ${BUILD_NUMBER} ${currentBuild.result}",
           from: "jenkins@bwg.com.br",
           body: "${currentBuild.result}: ${BUILD_URL}",
           attachLog: true,
           compressLog: true

        }

       failure {
           hangoutsNotify message: "❌ Deu Errado!\n⏰ Tempo de Duração: ${currentBuild.duration / 1000} segundos", token: "$CHAT_TOKEN", threadByJob: false 
        }
    }  
}