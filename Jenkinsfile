COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven3.9.6"
    }

    stages {
        stage ("Git clone") {
            steps {
                git branch: 'main', url: 'https://github.com/HenryKum23/web-app.git'
            }
        }

        stage ("Build with maven") {
            steps {
                sh "mvn clean"
            }
        }

        stage ("Run Test") {
            steps {
                sh "mvn test"
            }
        }

        stage ("maven package") {
            steps {
                sh "mvn package"
            }
        }

        stage ("sonarqube analysis") {
            environment {
                scannerHome = tool 'sonarHenry'
            }
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh '${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=HenryFriday'
                    }

                }
            }
        }

       stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage ("upload to nexus") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/My-Second-Pipeline-Practical/target/web-app.war', type: 'war']], credentialsId: 'Jenkins-Nexus', groupId: 'com.mt', nexusUrl: '3.85.204.21:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.1.2-RELEASE'
            }
        }

        stage ("Deploy to UAT") {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://44.202.63.231:8080')], contextPath: null, war: 'target/*.war'
            }
        }

    }

    post {
        success {
            slackSend channel: 'testing', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}"
        }
        failure {
            slackSend channel: 'testing', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}"
        }
    }
    
}