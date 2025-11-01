# Primer Job - Service Generator Pipeline

A Jenkins pipeline that automates the creation of new microservices from predefined templates (drizzle templates) with complete CI/CD setup.

## Overview

The Primer job streamlines the process of creating new services by:
- Cloning from language-specific templates
- Creating GitHub repositories with proper configuration
- Setting up webhooks for CI/CD integration
- Updating seed jobs for automatic pipeline generation


## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `SERVICE_NAME` | String | `test` | Name of the new service (lowercase alphanumeric only) |
| `GITHUB_ORG` | String | `meghdo-cloud` | GitHub organization where the repository will be created |
| `LANG` | Choice | - | Programming language template (`java`, `python`, `go`, `nodejs`, `angular`, `react`) |
| `NAMESPACE` | String | `byngtech` | Kubernetes namespace for service deployment |

## Usage

### Running the Job

1. Navigate to the Primer job in Jenkins
2. Click "Build with Parameters"
3. Fill in the required parameters:
   ```
   SERVICE_NAME: my-new-service
   GITHUB_ORG: your-org
   LANG: java
   NAMESPACE: production
   ```
4. Click "Build"

### Service Name Requirements

- Must start with a lowercase letter
- Can contain lowercase letters and numbers only
- No special characters, spaces, or uppercase letters
- Examples: ✅ `userservice`, `api2`, `dataprocessor` | ❌ `User-Service`, `api_v2`, `DataProcessor`

## Pipeline Stages

### Stage 1: Create a new Github Repo

**What it does:**
- Validates template existence for the selected language
- Validates service name format
- Checks if repository already exists
- Creates new private repository if it doesn't exist
- Clones template and customizes it for the new service
- Sets up GitHub webhook for Jenkins integration
- Pushes initial code to the new repository

**Template Customizations:**
- Replaces `drizzle{LANG}` with `SERVICE_NAME` throughout the codebase
- Updates namespace configuration from `default` to specified `NAMESPACE`
- Updates database service references to match namespace
- For Java projects: Renames package structure

### Stage 2: Update Seed Job

**What it does:**
- Clones the `jenkins-jobs` repository
- Adds the new repository SSH URL to `gitrepos.txt`
- Commits and pushes changes to trigger seed job execution
- Automatically creates Jenkins pipeline for the new service

## Environment Variables

| Variable | Description |
|----------|-------------|
| `GITHUB_API_URL` | GitHub API endpoint (`https://api.github.com`) |
| `WEBHOOK_URL` | Jenkins webhook URL for GitHub integration |
| `GITHUB_TOKEN` | GitHub personal access token (from credentials) |
| `SEED_JOB_REPO` | Repository containing Jenkins job definitions |

## Supported Languages & Templates

| Language | Template Repository | Description |
|----------|-------------------|-------------|
| `java` | `drizzlejava` | Spring Boot application template |
| `python` | `drizzlepython` | Python application template |
| `go` | `drizzlego` | Go application template |
| `nodejs` | `drizzlenodejs` | Node.js application template |
| `angular` | `drizzleangular` | Angular frontend template |
| `react` | `drizzlereact` | React frontend template |

## Error Handling

### Common Errors

1. **Template Not Found (404)**
   ```
   {LANG} - template hasnt be subscribed
   ```
   **Solution:** Ensure the template repository `drizzle{LANG}` exists in the organization

2. **Invalid Service Name**
   ```
   Invalid application format: '{SERVICE_NAME}' - special characters not allowed
   ```
   **Solution:** Use only lowercase letters and numbers, starting with a letter

3. **Repository Already Exists**
   ```
   Repository '{SERVICE_NAME}' already exists in the organization '{GITHUB_ORG}'. Skipping creation.
   ```
   **Solution:** Choose a different service name or delete the existing repository

## Post-Creation Steps

After successful execution:

1. **Verify Repository Creation:**
   - Check GitHub organization for the new repository
   - Verify webhook configuration in repository settings

2. **Jenkins Pipeline:**
   - Seed job will automatically create a new Jenkins pipeline
   - Pipeline will be available within a few minutes

3. **Development Setup:**
   - Clone the new repository locally
   - Follow language-specific setup instructions in the generated README
   - Make initial commits to trigger the CI/CD pipeline

## Troubleshooting

### Debug Information

- Check Jenkins console output for detailed error messages
- Verify GitHub token permissions include:
  - Repository creation
  - Webhook management
  - Organization access

### Manual Cleanup

If the job fails partway through:

1. **Remove Created Repository:**
   ```bash
   curl -X DELETE -H "Authorization: token {GITHUB_TOKEN}" \
     https://api.github.com/repos/{GITHUB_ORG}/{SERVICE_NAME}
   ```

2. **Remove from Seed Job:**
   - Edit `jenkins-jobs/seed_jobs/gitrepos.txt`
   - Remove the line containing the failed service
   - Commit and push changes

## Security Considerations

- GitHub token is stored securely in Jenkins credentials
- All created repositories are private by default
- Webhook URLs use HTTPS with proper SSL verification
- Service names are validated to prevent injection attacks

## Contributing

To add support for new languages:

1. Create a new template repository named `drizzle{LANG}`
2. Add the language choice to the `LANG` parameter in the Jenkinsfile
3. Test the pipeline with the new language option

## Support

For issues or questions:
- Check Jenkins console logs for detailed error information
- Verify all prerequisites are met
- Contact the DevOps team for GitHub organization access issues
