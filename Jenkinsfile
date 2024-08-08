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
                    image 'maven:3.9.4-eclipse-temurin-17-alpine'
                    args '-u root --network host'
              }
            }
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh 'mvn sonar:sonar -Dsonar.token=$SONARQUBE_CREDENTIALS_PSW -Dsonar.projectKey=WebGoat -Dsonar.qualitygate.wait=true -Dsonar.host.url=http://147.139.166.250:9009 -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco-unit-test-coverage-report/jacoco.xml' 
                }
            }
        }
    }
}