pipeline {
  agent {
    kubernetes {
      label 'maven'
      defaultContainer 'maven'
    }

  }
  stages {
    stage('get') {
      steps {
        script {
          withCredentials([kubeconfigFile(credentialsId: 'local', variable: 'config')]) {
            sh """
            export KUBECONFIG=\${config}
            kubectl get po --all-namespaces
            helm ls 
            """
          }
        }

      }
    }
    stage('@admin') {
      steps {
          timeout(time: 500, unit: 'SECONDS') {
            input(message: 'Ready to go?  ', submitter: 'admin')
          }
      }
    }
    stage('envregister') {
      steps {
        script {
          docker_host = "w564791"
          
          img_name = sh(returnStdout: true, script: 'basename  `git config --get remote.origin.url` .git').trim()
          build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
          LOCAL_RRANCH=sh(returnStdout: true,script: 'git rev-parse --abbrev-ref HEAD').trim()
          VERSION = readMavenPom().getVersion()
          build_tag = "${LOCAL_RRANCH}-${build_tag}-${VERSION}"
          docker_img_name = "${docker_host}/${img_name}:${build_tag}"

        }

        sh 'mvn package'
        script {
          docker.withRegistry('', 'dockerHubRegistry') {
          def customImage = docker.build("${docker_img_name}")
          customImage.push()
          }
        }

      }
    }
  }
  parameters {
    choice(name: 'BRANCH', choices: '''Development
release/release_QA
master''', description: 'Selct the branch to deploy to repective Airflow')
  }
      post {
        success {
            dingtalk (
                        robot: '044c145d-0df9-401c-b7f4-6eb5f8932498',
                        type: 'LINK',
                        title: 'build seucess',
                        text: [
                            '测试链接类型的消息',
                            '分行显示，哈哈哈哈'
                        ],
                        messageUrl: '${env.JENKINS_URL}',
                        picUrl: 'https://ae01.alicdn.com/kf/Hdfe28d2621434a1eb821ac6327b768e79.png'
                    )
        }
        failure {
            dingtalk (
                        robot: '044c145d-0df9-401c-b7f4-6eb5f8932498',
                        type: 'LINK',
                        title: 'build false',
                        text: [
                            '测试链接类型的消息',
                            '分行显示，哈哈哈哈'
                        ],
                        messageUrl: '${env.JENKINS_URL}',
                        picUrl: 'https://ae01.alicdn.com/kf/Hdfe28d2621434a1eb821ac6327b768e79.png'
                    )
        }
      }
}
