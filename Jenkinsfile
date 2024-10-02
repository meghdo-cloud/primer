pipeline {
    agent { label 'javaagent' }

    parameters {
        string(name: 'SERVICE_NAME', defaultValue: 'my-service', description: 'Name of the new service')
        string(name: 'GITHUB_ORG'), description: 'Git repo where the new application'
    }

    environment {
        APP_TEMP = 'https://github.com/meghdo-cloud/drizzle.git' 
        GITHUB_API_URL = 'https://api.github.com'
        GITHUB_TOKEN = credentials('git_admin_token')
    }

    stages {
        stage('Create a new Github Repo') {
            steps {
              script {
                git branch: 'main', url: "${env.APP_TEMP}"
                 sh """
                    curl -H "Authorization: token ${env.GITHUB_TOKEN}" -d '{"name": "${params.SERVICE_NAME}", "private": true}' ${env.GITHUB_API_URL}/orgs/${GITHUB_ORG}/repos
                    find . -type f -exec sed -i 's/drizzle/${params.SERVICE_NAME}/g' {} +
                    git init
                    git remote add origin ${env.NEW_REPO_URL}
                    git add .
                    git commit -m "Initialize new service: ${params.SERVICE_NAME}"
                    git push -u origin main                    
                    """
              }
            }
        }
        
    post {
        always {
            cleanWs() // Clean workspace after job finishes
        }
    }
}
