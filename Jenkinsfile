import groovy.json.JsonOutput

pipeline {
    agent {
        docker {
            image 'node:18'
        }
    }
    environment {
        N8N_WEBHOOK_URL = 'http://192.168.48.132:5678/webhook-test/jenkins-discord'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/pratham7289/N8N-Test.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                echo 'Building Node.js app...'
                sh 'node index.js & sleep 5; kill $!'
            }
        }
    }
    post {
        always {
            script {
                def buildStatus = currentBuild.currentResult ?: 'UNKNOWN'
                def failureReason = buildStatus == 'SUCCESS' ? '' : currentBuild.rawBuild.getLog(100).join('\n').split('\n').findAll { it.contains('ERROR') || it.contains('Failed') }.join('; ') ?: 'Unknown error'
                def message = buildStatus == 'SUCCESS' ?
                    "✅ Build #${env.BUILD_NUMBER} for '${env.JOB_NAME}' succeeded! Ready for demo. 🚀" :
                    "❌ Build #${env.BUILD_NUMBER} for '${env.JOB_NAME}' failed! Reason: ${failureReason}\n📞 Contact DevOps team now!"
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
