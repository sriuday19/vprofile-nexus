def COLOR_MAP = [
    'SUCCESS' : 'good',
    'FAILURE' : 'danger',
]
pipeline {
    agent any

    tools {
        jdk "OracleJDK8"
        maven "MAVEN3"
    }

    environment {
        scannerHome = tool 'sonar4'
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.82.67:8081"
        NEXUS_REPOSITORY = "vprofile-repo"
        NEXUS_CREDENTIAL_ID = "nexus-token"
    }

    stages {

        stage('fecth code') {

            steps {
                git branch:'main', url: 'https://github.com/sriuday19/vprofile-ci.git'
            }
        }

        stage('Build code') {

            steps {
                sh 'mvn clean install'
            } 

            post {
                success {
                    echo 'Archieving the artifact'
                    archiveArtifacts artifacts: "**/target/*.war"
                }
            }
        }

        stage('Test') {

            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {

            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Code Analysis withe Sonqrqube') {

            steps {

                withSonarQubeEnv("sonar") {
                    sh "${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-app \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml "
                    
                }
            }
        }

        stage('Quality-gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('uploading the artifact to nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: 'QA',
                            version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: 'vprofile-app',
                                classifier: '',
                                file: 'target/vprofile-v2.war',
                                type: 'war'],
                            ]
                        );
                }
            }
        }
    }
    post {
       always {
          echo "slack notification"
          slackSend(
            channel: '#vprofile-app',
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* - job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at ${env.BUILD_URL}"
          )
       }
    }
}