pipeline {
  agent {
    docker {
      image 'maven:3-alpine'
      args '-v /root/.m2:/root/.m2'
    }

  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
    }
    stage('Test') {
      post {
        always {
          junit 'target/surefire-reports/*.xml'

        }

      }
      steps {
        sh 'mvn test'
      }
    }
    stage('SonarQube analysis') {
      steps {
        script {
          scannerHome = tool 'sonarTool'
          echo "${scannerHome}"
        }

        withSonarQubeEnv('sonar') {
          sh 'mvn sonar:sonar'
        }

      }
    }
    stage('SonarQube Gatekeeper') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          script {
            def qualitygate = waitForQualityGate()
            if (qualitygate.status != "OK") {
              error "Pipeline aborted due to quality gate coverage failure: ${qualitygate.status}"
            }
          }

        }

      }
    }
    stage('Deploy') {
      parallel {
        stage('Deploy Stage') {
          when {
            branch 'stage'
          }
          steps {
            sh './jenkins/scripts/deliver.sh'
          }
        }
        stage('Deploy Prod') {
          steps {
            sh 'sh \'./jenkins/scripts/deliver.sh\''
            script {
              def deploy= input(message: 'Deploy to Prod?', ok: 'Yes', parameters: [booleanParam(defaultValue: true,
              description: 'If you like Java, just push the button',name: 'Yes?')])
            }

          }
        }
      }
    }
  }
  options {
    skipStagesAfterUnstable()
  }
}