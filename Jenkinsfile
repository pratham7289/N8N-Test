import groovy.json.JsonOutput

pipeline {
    agent any
    environment {
        // n8n Webhook URL
        N8N_WEBHOOK_URL = 'http://192.168.48.132:5678/webhook-test/jenkins-discord'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Running build...'
                // Replace with actual build commands
                sh 'echo Build success' // simulate success
                // sh 'exit 1'           // simulate failure
            }
        }
    }

    post {
        always {
            script {
                // Determine build status
                def buildStatus = currentBuild.currentResult ?: 'UNKNOWN'
                
                // Message for Discord
                def message = buildStatus == 'SUCCESS' ? 
                    "🚀 Build #${env.BUILD_NUMBER} for '${env.JOB_NAME}' succeeded!" : 
                    "❌ Build #${env.BUILD_NUMBER} for '${env.JOB_NAME}' failed! Contact DevOps or Developer."

                // Payload to send to n8n
                def payload = [
                    job_name    : env.JOB_NAME ?: 'unknown',
                    build_number: env.BUILD_NUMBER ?: '0',
                    status      : buildStatus,
                    url         : env.BUILD_URL ?: '#',
                    message     : message
                ]

                def jsonPayload = JsonOutput.toJson(payload)
                echo "Payload: ${jsonPayload}"

                // Send POST request to n8n
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
