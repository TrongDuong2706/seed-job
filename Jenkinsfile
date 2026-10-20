pipeline {
    agent {
        label 'built-in'
    }

    options {
        disableConcurrentBuilds()
        skipDefaultCheckout(true)
        timestamps()
    }

    parameters {
        string(
            name: 'CENTRAL_REPO_URL',
            defaultValue: 'https://github.com/TrongDuong2706/centralized-jenkins-file.git',
            description: 'Git repository that contains centralized Jenkinsfiles.'
        )
        string(
            name: 'CENTRAL_REPO_CREDENTIALS_ID',
            defaultValue: '',
            description: 'Credential used to read the central Jenkinsfile repository. Leave empty for a public repository.'
        )
        string(
            name: 'CENTRAL_REPO_BRANCH',
            defaultValue: 'main',
            description: 'Branch containing the centralized Jenkinsfiles.'
        )
    }

    stages {
        stage('Checkout Seed Repository') {
            steps {
                checkout scm
            }
        }

        stage('Checkout Centralized Config') {
            steps {
                script {
                    def remote = [url: params.CENTRAL_REPO_URL]
                    if (params.CENTRAL_REPO_CREDENTIALS_ID?.trim()) {
                        remote.credentialsId = params.CENTRAL_REPO_CREDENTIALS_ID.trim()
                    }

                    dir('centralized-jenkins-file') {
                        checkout([
                            $class: 'GitSCM',
                            branches: [[name: "*/${params.CENTRAL_REPO_BRANCH}"]],
                            extensions: [],
                            userRemoteConfigs: [remote]
                        ])
                    }
                }
            }
        }

        stage('Generate Pipeline Jobs') {
            steps {
                jobDsl(
                    targets: 'seed/generateJobs.groovy',
                    removedJobAction: 'DISABLE',
                    removedViewAction: 'IGNORE',
                    lookupStrategy: 'JENKINS_ROOT',
                    sandbox: true,
                    additionalParameters: [
                        centralRepoUrl: params.CENTRAL_REPO_URL,
                        centralRepoCredentialsId: params.CENTRAL_REPO_CREDENTIALS_ID,
                        centralRepoBranch: params.CENTRAL_REPO_BRANCH
                    ]
                )
            }
        }
    }
}
