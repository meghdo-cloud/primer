pipeline {
    agent { label 'jnlpagent' }

    parameters {
        string(name: 'SERVICE_NAME', defaultValue: 'test', description: 'Name of the new service')
        string(name: 'GITHUB_ORG', defaultValue: 'meghdo-cloud', description: 'Git repo where the new application')
        choice(name: 'LANG', choices: ['java','python','go','nodejs','angular','react'])
    }

    environment {
        GITHUB_API_URL = 'https://api.github.com'
        WEBHOOK_URL = 'https://jenkins.meghdo.cloud/github-webhook/'
        GITHUB_TOKEN = credentials('git_admin_token')
        SEED_JOB_REPO = 'jenkins-jobs'
    }

    stages {
        stage('Create a new Github Repo') {
            steps {
              script {
                //set the env app template
                env.APP_TEMP = "https://github.com/${params.GITHUB_ORG}/drizzle${params.LANG}.git"
                env.CLONE_URL = "https://${env.GITHUB_TOKEN}@github.com/${params.GITHUB_ORG}/drizzle${params.LANG}.git"

                //check if the template exists
                deftemplateexists = sh(
                    script: """
                        curl -s -o /dev/null -w "%{http_code}" -H "Authorization: token ${env.GITHUB_TOKEN}" \
                        ${env.APP_TEMP}
                    """,
                    returnStdout: true
                ).trim()

                if (deftemplateexists == '404') {
                    echo "${params.LANG} - template hasnt be subscribed"
                    System.exit(1) 
                }   

                if (!("${params.SERVICE_NAME}" =~ /^[a-z]+[a-z0-9]*$/)) {
                        error "Invalid application format: '${params.SERVICE_NAME}' - special characters not allowed" }
                   
                    // Create a temporary directory for the template
                    sh "mkdir -p template_files"
                    dir('template_files') {
                        // Clone template and copy files without Git history
                        sh """
                            git clone --depth 1 ${env.CLONE_URL} .
                            rm -rf .git
                        """
                    }      
                
                
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
                    curl -H "Authorization: token ${env.GITHUB_TOKEN}" -H "Content-Type: application/json" -X POST \
                                 -d '{
                                        "name": "web",
                                        "active": true,
                                        "events": ["push", "pull_request"],
                                        "config": {
                                            "url": "${env.WEBHOOK_URL}",
                                            "content_type": "json",
                                            "insecure_ssl": "0"
                                        }
                                     }' \
                                 ${env.GITHUB_API_URL}/repos/${env.GITHUB_ORG}/${params.SERVICE_NAME}/hooks
                    cd template_files
                    if [ "${params.LANG}" = "java" ]; then
                        mv ./src/main/java/cloud/meghdo/drizzle ./src/main/java/cloud/meghdo/${params.SERVICE_NAME} 
                    fi
                    find . -type f -exec sed -i 's/drizzle${params.LANG}/${params.SERVICE_NAME}/g' {} +
                    git init
                    git config user.name "${env.GIT_USER_NAME}"
                    git config user.email "${env.GIT_USER_EMAIL}"
                    git add .
                    git commit -m "Initialize new service: ${params.SERVICE_NAME}"
                    git branch -M main
                    git remote add origin https://${env.GITHUB_TOKEN}@github.com/${env.GITHUB_ORG}/${params.SERVICE_NAME}.git
                    git push -u origin main
                    """
                }
              }
            }
        }
        stage('Update Seed Job') {
         steps {
            script {
                // Get the SSH URL of the newly created repo
                def sshUrl = "git@github.com:${env.GITHUB_ORG}/${params.SERVICE_NAME}.git"

                sh """
                git clone https://$GITHUB_TOKEN@github.com/${env.GITHUB_ORG}/${env.SEED_JOB_REPO}.git
                
                """


                // Append the new repo SSH URL to gitrepos.txt
                sh """
                    echo '${sshUrl}' >> ./jenkins-jobs/seed_jobs/gitrepos.txt
                    cat ./jenkins-jobs/seed_jobs/gitrepos.txt
                """

                // Commit and push the changes
               sh """
                    cd jenkins-jobs/seed_jobs
                    git config user.email "jenkins@${env.GITHUB_ORG}.com"
                    git config user.name "Jenkins CI"
                    git add gitrepos.txt
                    git commit -m "Added ${params.SERVICE_NAME} repo to gitrepos.txt"
                    git push https://$GITHUB_TOKEN@github.com/${env.GITHUB_ORG}/${env.SEED_JOB_REPO}.git main
               """

                echo "Seed job repo updated successfully!"
                
               }
            }
        }
    }
}
