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
          sh "${scannerHome}/bin/sonar-scanner"
        }

      }
    }
    stage ('SonarQube Gatekeeper') {
        def qualitygate = waitForQualityGate()
        if (qualitygate.status != "OK") {
         error "Pipeline aborted due to quality gate coverage failure: ${qualitygate.status}"
        }

    }
    stage('Deliver') {
      when {
        branch 'stage'
      }
      steps {
        sh './jenkins/scripts/deliver.sh'
      }
    }
  }
  options {
    skipStagesAfterUnstable()
  }
}