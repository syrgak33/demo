pipeline {
    agent any
      environment {
        TEST_BRANCH = 'test'  // Specify the name of your test branch
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from the GitHub repository
                git branch: 'test',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/syrgak33/demo.git'
               // git credentialsId: 'github-creds', url: 'https://github.com/syrgak33/demo.git'
            }
        }
        
        stage('Build and Test') {
            when {
                // Run this stage only if changes are detected in the specified branch
                expression { env.BRANCH_NAME == env.TEST_BRANCH || env.CHANGE_TARGET == env.TEST_BRANCH }
            }
            steps {
                // Your build and test commands here
                sh 'mvn clean test' // Example for Maven project, adjust as needed
            }
            post {
                always {
                    // Publish JUnit test results
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Deploy to Test Environment') {
            when {
                // Run this stage only if changes are detected in the specified branch
                expression { env.BRANCH_NAME == env.TEST_BRANCH || env.CHANGE_TARGET == env.TEST_BRANCH }
            }
            steps {
                // Your deployment commands for the test environment
                sh 'echo "Deploying to test environment"'
                // Add your deployment steps here
            }
        }
    }

    
    post {
        success {
            script {
                // Update GitHub pull request status on successful build and deployment
                updateGitHubPRStatus('success', 'Jenkins build and deployment to test environment passed!')
            }
        }
        failure {
            script {
                // Update GitHub pull request status on build or deployment failure
                updateGitHubPRStatus('failure', 'Jenkins build or deployment to test environment failed!')
            }
        }
    }
}

def updateGitHubPRStatus(state, description) {
    def pullRequestId = env.CHANGE_ID // Get the pull request ID from environment variables
    def githubToken = credentials('github-token') // Use Jenkins credentials to retrieve the GitHub token
    
    def payload = [
        state: state,
        description: description,
        context: 'Jenkins CI'
    ]
    
    httpRequest(
        contentType: 'APPLICATION_JSON',
        httpMode: 'POST',
        url: "https://api.github.com/repos/syrgak33/demo/statuses/$pullRequestId",
        authentication: githubToken,
        requestBody: groovy.json.JsonOutput.toJson(payload)
    )
}
