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
        stage('Secret Scanning Using Trufflehog') {
            agent {
                docker {
                    image 'trufflesecurity/trufflehog:latest'
                    args '-u root --entrypoint='
                }
            }
            steps {
                sh 'trufflehog filesystem . --exclude-paths trufflehog-excluded-paths.txt --json > trufflehog-scan-result.json'
                sh 'cat trufflehog-scan-result.json'
                archiveArtifacts artifacts: 'trufflehog-scan-result.json'
            }
        }
        stage('Build Maven') {
            agent {
                docker {
                    image 'maven:3.9.4-eclipse-temurin-17-alpine'
                    args '-u root'
                }
            }
            steps {
                sh 'mvn -B -DskipTests clean package'
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
                    sh 'sonar-scanner -Dsonar.projectKey=javulna -Dsonar.qualitygate.wait=true -Dsonar.sources=. -Dsonar.host.url=http://147.139.166.250:9009 -Dsonar.token=$SONARQUBE_CREDENTIALS_PSW' 
                }
            }
        }
    }
}