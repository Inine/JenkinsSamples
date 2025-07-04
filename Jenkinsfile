pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch name')
        string(name: 'APP_VERSION', defaultValue: 'latest', description: 'Application version to deploy')
        string(name: 'MY_SECRET_TOKEN', defaultValue: 'jenkins_creds')
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            branchFilter: 'origin/(.*)',
            defaultValue: 'master',
            selectedValue: 'NONE',
            sortMode: 'ASCENDING',
            useRepository: 'https://github.com/docker/awesome-compose.git'
        )
    }
    
    environment {
        APP_ENV = "${DEPLOY_ENV}"
        APP_VER = "${APP_VERSION}"
        RUN_TST = "${RUN_TESTS}"
        TARGET_DIR = "target${env.APP_ENV}_${env.APP_VERSION}"
        TOKEN="${MY_SECRET_TOKEN}"
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    echo "Building project from branch ${params.BRANCH_NAME}..."
                    withCredentials([usernamePassword(
                      credentialsId: "${env.TOKEN}",
                      usernameVariable: 'USERNAME',
                      passwordVariable: 'PASSWORD'
                    )]) {
                      sh 'echo "$USERNAME:$PASSWORD something hidden here"'
                    }
                    sh """
                        mkdir -p ${env.TARGET_DIR}
                        echo 'fake jar content' > ${env.TARGET_DIR}/app.jar
                    """
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: "${env.TARGET_DIR}/*.jar", fingerprint: true
                    echo "Artifact build completed successfully"
                }
                failure {
                    echo "Build failed"
                }
            }
        }

        stage('Testing') {
            when {
                expression { return env.RUN_TST }
            }
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo "Running unit tests"
                        sh """
                        echo "Running unit tests"
                        mkdir -p ${env.TARGET_DIR}/surefire-reports
                        touch ${env.TARGET_DIR}/surefire-reports/test.xml
                        """
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo "Running integration tests"
                        sh """
                        echo "Running integration tests"
                        mkdir -p ${env.TARGET_DIR}/failsafe-reports
                        touch ${env.TARGET_DIR}/failsafe-reports/it.xml
                        """
                    }
                }
                stage('Code Quality') {
                    steps {
                        echo "Analyzing code quality"
                        sh 'echo "Code analysis done"'
                    }
                }
            }
        }

        stage('Deploy to Test') {
            when {
                anyOf {
                    expression { return params.ENVIRONMENT == 'dev' }
                    expression { return params.ENVIRONMENT == 'test' }
                }
            }
            steps {
                echo "Deploying to ${params.ENVIRONMENT} environment..."
                sh 'echo "Deploying to dev/test server..."'
            }
        }

        stage('Deploy to Prod') {
            when {
                expression { return params.ENVIRONMENT == 'prod' }
            }
            steps {
                input message: 'Confirm deployment to production', ok: 'Deploy'
                echo "Deploying to production..."
                sh 'echo "Deploying to prod server..."'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
        always {
            deleteDir()
        }
    }
}
