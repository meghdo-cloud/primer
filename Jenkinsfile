pipeline {
    agent { label 'javaagent' }

    parameters {
        string(name: 'SERVICE_NAME', defaultValue: 'test', description: 'Name of the new service')
        string(name: 'GITHUB_ORG', defaultValue: 'meghdo-cloud', description: 'Git repo where the new application')
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
                if (!("${params.SERVICE_NAME}" =~ /^[a-z]+[a-z0-9-]*$/)) {
                        error "Invalid application format: '${params.SERVICE_NAME}' - special characters not allowed" }  
                git branch: 'main', url: "${env.APP_TEMP}"
                def repoExistsResponse = sh(
                    script: """
                        curl -s -o /dev/null -w "%{http_code}" -H "Authorization: token ${env.GITHUB_TOKEN}" \
                        ${env.GITHUB_API_URL}/repos/${env.GITHUB_ORG}/${params.SERVICE_NAME}
                    """,
                    returnStdout: true
                ).trim()

                // If the response is 200, the repo exists
                if (repoExistsResponse == '200') {
                    echo "Repository '${params.SERVICE_NAME}' already exists in the organization '${env.GITHUB_ORG}'. Skipping creation."
                } else if (repoExistsResponse == '404') {
                    sh """
                    curl -H "Authorization: token ${env.GITHUB_TOKEN}" -d '{"name": "${params.SERVICE_NAME}", "private": true}' ${env.GITHUB_API_URL}/orgs/${GITHUB_ORG}/repos
                    pwd
                    mv ./src/main/java/cloud/meghdo/drizzle/drizzleApplication.java ./src/main/java/cloud/meghdo/drizzle/${params.SERVICE_NAME}Application.java
                    mv ./src/main/java/cloud/meghdo/drizzle ./src/main/java/cloud/meghdo/${params.SERVICE_NAME}
                    find . -type f -exec sed -i 's/drizzle/${params.SERVICE_NAME}/g' {} +
                    git config user.name "${env.GIT_USER_NAME}"
                    git config user.email "${env.GIT_USER_EMAIL}"
                    rm -f .git/index
                    git reset
                    git remote remove origin
                    git remote add origin https://${env.GITHUB_TOKEN}@github.com/${env.GITHUB_ORG}/${params.SERVICE_NAME}.git
                    git add .
                    git commit -m "Initialize new service: ${params.SERVICE_NAME}"
                    git push -u origin main                    
                    """
                }
              }
            }
        }
  }      
}
