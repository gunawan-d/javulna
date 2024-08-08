pipeline {
    agent any
    environment {
        SONARQUBE_CREDENTIALS_PSW = credentials('SONARQUBE_CREDENTIALS_PSW')
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('SAST SonarQube') {
            agent {
              docker {
                  image 'sonarsource/sonar-scanner-cli:latest'
                  args '--network host -v ".:/usr/src" --entrypoint='
              }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'sonar-scanner -Dsonar.projectKey=nodejs-goof -Dsonar.qualitygate.wait=true -Dsonar.sources=. -Dsonar.host.url=http://147.139.166.250:9009 -Dsonar.token=$SONARQUBE_CREDENTIALS_PSW -Dsonar.scanner.dumpToFile=sonar-report.json'
                }
                archiveArtifacts artifacts: 'sonar-report.json'
            }
        }
    }
}