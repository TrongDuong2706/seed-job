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
                script {
                    def configFiles = [
                        'centralized-jenkins-file/todolist/config.yaml',
                        'centralized-jenkins-file/crm/config.yaml',
                        'centralized-jenkins-file/ecommerce/config.yaml'
                    ]
                    def pipelineConfigs = []

                    configFiles.each { configFile ->
                        if (!fileExists(configFile)) {
                            echo "Skip ${configFile}: file does not exist."
                            return
                        }

                        def document = readYaml file: configFile
                        def projectFolder = (document?.folder ?: '').toString().trim()
                        (document?.pipelines ?: []).each { pipelineConfig ->
                            pipelineConfigs << (pipelineConfig + [
                                folder    : pipelineConfig.folder ?: projectFolder,
                                seedSource: configFile
                            ])
                        }
                    }

                    writeJSON file: 'seed/pipelines.json', json: pipelineConfigs, pretty: 2
                }

                jobDsl(
                    targets: 'seed/generateJobs.groovy',
                    removedJobAction: 'DISABLE',
                    removedViewAction: 'IGNORE',
                    lookupStrategy: 'JENKINS_ROOT',
                    sandbox: false,
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
