def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any

    tools {
        jdk "JDK21"
        maven "Maven3.9"
    }

    environment {
        SONAR_TOKEN = credentials('SonarToken')
        SONAR_HOST = 'http://sonarqube:9000'
    }

    stages {

        stage('Check Java') {
            steps {
                sh 'java -version'
            }
        }

        stage('Fetch Source Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Marko-devops/Test_doker'
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'Sonar6.2'
            }
            steps {
                withSonarQubeEnv('Sonarserver') {
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=Jenkins_pipeline \
                        -Dsonar.projectName="Test Project" \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=.
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -B'
            }
        }

        stage("Upload Artifact to Nexus") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.18.0.4:8081',          // IP Nexusa iz docker networka
                    groupId: 'com.example',
                    version: '1.0.0-SNAPSHOT',
                    repository: 'Jenkins_artf', // snapshot repo
                    credentialsId: 'Nexuslogin',           // Jenkins credential ID (username/password)
                    artifacts: [
                        [artifactId: 'test-web-app',
                         classifier: '',
                         file: 'target/test-web-app-1.0.0-SNAPSHOT.jar',
                         type: 'jar']
                    ]
                )
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#new-channel',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}