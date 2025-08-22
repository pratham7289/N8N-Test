import groovy.json.JsonOutput

pipeline {
    agent any
    environment {
        N8N_WEBHOOK_URL = 'http://192.168.48.132:5678/webhook-test/jenkins-discord'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/pratham7289/N8N-Test.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Running build...'
                sh 'echo Build success'
            }
        }
    }
    post {
        always {
            script {
                def buildStatus = currentBuild.currentResult ?: 'UNKNOWN'
                def message = buildStatus == 'SUCCESS' ?
                    "🚀 Build #${env.BUILD_NUMBER} for '${env.JOB_NAME}' succeeded!" :
                    "❌ Build #${env.BUILD_NUMBER} for '${env.JOB_NAME}' failed! Contact DevOps Team"
                def payload = [
                    job_name    : env.JOB_NAME ?: 'unknown',
                    build_number: env.BUILD_NUMBER ?: '0',
                    status      : buildStatus,
                    url         : env.BUILD_URL ?: '#',
                    message     : message
                ]
                def jsonPayload = JsonOutput.toJson(payload)
                echo "Payload: ${jsonPayload}"
                try {
                    httpRequest(
                        httpMode: 'POST',
                        contentType: 'APPLICATION_JSON',
                        url: "${env.N8N_WEBHOOK_URL}",
                        requestBody: jsonPayload
                    )
                } catch (Exception e) {
                    echo "Error sending to n8n: ${e.message}"
                }
            }
        }
    }
}
