#!/usr/bin/env groovy
import groovy.json.JsonOutput

pipeline {
  agent {
    label "mesos"
  }

  triggers {
    cron('0 5 * * *')
  }

  environment {
    proxyUrl = "http://MyProxy.example.com:443"
    apiMarathon = ".mesos.example.com/service/marathon/v2/apps"
    def lastVersion = getLastVersion()
    def currentVersion = readFile('docker/CURRENT_VERSION')
  }

  stages {
    stage("Init") {
      steps {
        deleteDir()
        checkout scm
      }
    }

    stage("Check Version") {
      steps {
        script {
          if (env.currentVersion == lastVersion) {
            currentBuild.result = 'ABORTED'
            sendKafkaEvent('#B5B5B5', 'Não existe versão superior a atual ' + currentVersion + '!!!')
            error('Não existe versão superior a atual ' + currentVersion + '!!!')
          }
        }
      }
    }

    stage("Download & Prepare package") {
      steps {
        script {
          dir("./docker") {
            echo 'Baixando a versão ' + lastVersion + ' . Versão atual é a ' + currentVersion
              sh "echo 'Baixando o pacote !' \
                ; curl -LO -x ${proxyUrl} https://github.com/Landoop/schema-registry-ui/releases/download/v.${lastVersion}/schema-registry-ui-${lastVersion}.tar.gz \
                ; echo 'Preparando o pacote' \
                ; tar xf schema-registry-ui-package-${lastVersion}.tar.gz -C ./ \
                ; echo 'Limpando os arquivos antigos' \
                ; rm -rf schema-registry-ui/ \
                ; echo 'Criando a estrutura do schema-registry' \
                ; mkdir schema-registry-ui \
                ; tar xf schema-registry-ui-${lastVersion}.tar.gz -C schema-registry-ui/ \
                ; echo 'Criando o pacote tarball' \
                ; tar czf schema-registry-ui-package-${lastVersion}.tar.gz schema-registry-ui/ caddy/ run.sh \
                ; echo 'Removendo os arquivos temporários' \
                ; rm -rf schema-registry-ui-${lastVersion}.tar.gz schema-registry-ui caddy run.sh \
                ; echo 'Atualizando a versão corrente no arquivo CURRENT_VERSION' \
                ; echo ${lastVersion} > CURRENT_VERSION \
                ; git add . \
                ; git commit -m 'Atualizando a versão para a ${lastVersion}' \
                ; git push -u origin master"
          }
        }
      }
    }

    stage("Build Docker") {
      steps {
        script {
          dir("./docker") {
            docker.withRegistry("https://hub.example.com", "s_kafka_docker_account") {
              readFile(file: "Dockerfile").replaceAll("__LAST_VERSION__", "${lastVersion}")
              docker.build("kafka/schema-registry-ui:${lastVersion}-${BUILD_NUMBER}").push()
            }
          }
        }
      }
    }

    stage("Deploy QA - AWS") {
      steps {
        script {
          dir("./deploy") {
            def envMesos = 'QA'
            def url = "https://" + envMesos + apiMarathon
            def jsonMarathon = readFile(file: "${envMesos}/schema-registry-ui.json").replaceAll("__BUILD_NUMBER__", "${BUILD_NUMBER}")
            readFile(file: "${envMesos}/schema-registry-ui.json").replaceAll("__LAST_VERSION__", "${lastVersion}")
            def appId = "qa/kafka/schema-registry-ui"
            marathon(url, envMesos, jsonMarathon, appId)
            sendKafkaEvent('good', 'Deploy da app ' + appId + ' na versão ' + lastVersion + ' em ' + envMesos + ' com sucesso !!!')
          }
        }
      }
    }

    stage("Approval PRD") {
      steps {
        script {
          timeout(time: 12, unit: "HOURS") {
              sendKafkaEvent('warning', 'Aguardando aprovação para deploy em PRD !!!')
              input(message: "Approve Deployment?")
          }
        }
      }
    }

    stage("Deploy PRD - AWS") {
      steps {
        script {
          dir("./deploy") {
            def envMesos = 'GT'
            def url = "https://" + envMesos + apiMarathon
            def jsonMarathon = readFile(file: "${envMesos}/schema-registry-ui.json").replaceAll("__BUILD_NUMBER__", "${BUILD_NUMBER}")
            readFile(file: "${envMesos}/schema-registry-ui.json").replaceAll("__LAST_VERSION__", "${lastVersion}")
            def appId = "kafka/schema-registry-ui"
            marathon(url, envMesos, jsonMarathon, appId)
            sendKafkaEvent('good', 'Deploy da app ' + appId + ' na versão ' + lastVersion + ' em ' + envMesos + ' com sucesso !!!')
          }
        }
      }
    }

    stage("Deploy PRD - GCP") {
      steps {
        script {
          dir("./deploy") {
            def envMesos = 'TB'
            def url = "https://" + envMesos + apiMarathon
            def jsonMarathon = readFile(file: "${envMesos}/schema-registry-ui.json").replaceAll("__BUILD_NUMBER__", "${BUILD_NUMBER}")
            readFile(file: "${envMesos}/schema-registry-ui.json").replaceAll("__LAST_VERSION__", "${lastVersion}")
            def appId = "kafka/schema-registry-ui"
            marathon(url, envMesos, jsonMarathon, appId)
            sendKafkaEvent('good', 'Deploy da app ' + appId + ' na versão ' + lastVersion + ' em ' + envMesos + ' com sucesso !!!')
          }
        }
      }
    }

    stage("Clean") {
      steps {
        deleteDir()
      }
    }

  }
}

def getLastVersion() {
  return sh(
    script: "\
      ; curl -Ls -o /dev/null -x ${proxyUrl} -w %{url_effective} https://github.com/Landoop/schema-registry-ui/releases/latest | sed 's/.*tag\\/v// ; s/^\\.//'",
    returnStdout: true
  )
}

def marathon(url, envMesos, jsonMarathon, appId) {
  println("Efetuando o PUT da App " + appId + "!!!")
  try {
      httpRequest(
        contentType: "APPLICATION_JSON",
        httpMode: "PUT",
        requestBody: jsonMarathon,
        url: url + "/" + appId,
        validResponseCodes: "200:201"
    )
  } catch (err) {
    sendKafkaEvent('danger', 'Erro ao efetuar o deploy da App ' + appId + '. Mensagem de erro: ' + err)
  }
}

def sendKafkaEvent(status, message) {
    notifySlack("", [[
      "title"      : "Notification Log, build #${env.BUILD_NUMBER}",
      "title_link" : "${env.BUILD_URL}",
      "color"      : "${status}",
      "author_name": "Kafka Events",
      "thumb_url"  : "https://www.confluent.io/wp-content/uploads/2016/08/kafka-logo-wide-1.png",
      "text"       : "${message}",
      "mrkdwn_in"  : ["fields"],
      "fields"     : [
        [
          "title": "Author",
          "value": "Basileia Team",
          "short": true
        ],
        [
          "title": "Branch",
          "value": "${env.GIT_BRANCH}",
          "short": true
        ],
        [
          "title": "Last Commit",
          "value": "${message}",
          "short": false
        ]
      ]
  ]])
}

def notifySlack(text, attachments) {
    def slackURL = 'https://hooks.slack.com/services/MyTokens'
    def payload = JsonOutput.toJson([text       : text,
                                     channel    : "#kafka-events",
                                     attachments: attachments
    ])

    sh "curl -X POST -x ${proxyUrl} --data-urlencode \'payload=${payload}\' ${slackURL}"
}