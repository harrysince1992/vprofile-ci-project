def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools{
        maven "maven3"
        jdk "jdk8"
    }

    // Variables
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUSIP = '172.31.35.91'
        NEXUSPORT = '8081'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
    }
// Build stage
    stages{
        stage('Build'){
            steps{
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
// Test stage
        stage('Test'){
            steps{
                sh 'mvn -s settings.xml test'
            }
        }
// archive stage
        stage('Checkstyle analysis'){
            steps{
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

// Sonarscanner stage
        stage('Sonarqube analysis'){
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }

            steps{
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

// quality gate stage

        stage('quality gate'){
            steps{
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
            }
            }
        }

        // Artifact upload stage
        stage('push artifact to repo'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: "target/vprofile-v2.war",
                        type: 'war']
                    ]
                )
            }
        }
    }

    post {
        always {
            echo 'Slack notifications.'
            slackSend channel: "#devopscicd",
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "**${currentBuild.currentResult}**: Job ${env.JOB_NAME} build: ${env.BUILD_NUMBER}. \n More info at ${env.BUILD_URL}"
        }
    }
}