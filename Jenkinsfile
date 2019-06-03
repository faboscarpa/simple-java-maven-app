pipeline {
    agent {
        docker {
            image 'maven:3-alpine'
            args '-v /root/.m2:/root/.m2'
        }
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('SonarQube analysis') {
                withSonarQubeEnv('sonar') {
                  mvn sonar:sonar
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
}