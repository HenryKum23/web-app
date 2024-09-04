COLOR_MAP = [
  'SUCCESS': 'good',
  'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "maven"
    }
    stages {
        stage('git clone') {
            steps {
               git branch: 'main', url: 'https://github.com/HenryKum23/web-app.git'
            }
        }
        
        stage('build') {
            steps{
                sh 'mvn clean'
            }
        }
        
        stage('package') {
            steps{
                sh 'mvn package'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                ScannerHome = tool 'Sonarqubescan'
            }
            steps{
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh '${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=henry'
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps{
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage('Uplaod to Nexus') {
          steps{
            nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/var/lib/jenkins/workspace/Mawulorm/target/web-app.war', type: 'war']], credentialsId: 'Nexus ID', groupId: 'com.mt', nexusUrl: '18.119.136.78:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.1.2-RELEASE'
          }
        }

        stage('UAT') {
          steps{
            deploy adapters: [tomcat9(credentialsId: 'Tomcat', path: '', url: 'http://18.219.223.185:8080/')], contextPath: null, war: 'target/*.war'
          }
        }
    }

    post {
        success {
            slackSend(channel: '#team6-africa', color: 'good', message: "Build successful: ${currentBuild.fullDisplayName}")
        }
        failure {
            slackSend(channel: '#team6-africa', color: 'danger', message: "Build failed: ${currentBuild.fullDisplayName}")
        }
    }

} 