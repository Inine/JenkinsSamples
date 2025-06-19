pipeline {
    agent any

    options {
        timeout(time: 1, unit: 'HOURS')
        disableConcurrentBuilds()
    }

    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Git branch name')
        string(name: 'APP_VERSION', defaultValue: 'latest', description: 'Application version to deploy')
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Deployment environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        gitParameter(
            name: 'BRANCH',
            type: 'PT_BRANCH',
            branchFilter: 'origin/(.*)',
            defaultValue: 'master',
            selectedValue: 'NONE',
            sortMode: 'ASCENDING',
            useRepository: 'sparkjava-mysql/backend'
        )
    }
    
    environment {
        APP_ENV = "${DEPLOY_ENV}"
        APP_VER = "${APP_VERSION}"
        RUN_TST = "${RUN_TESTS}"
        TARGET_DIR = "target" + "${env.APP_ENV}" + "_" + "${env.APP_VERSION}"
    }
    
    stages {
        stage('Build') {
            steps {
                script {
                    echo "Building project from branch ${params.BRANCH_NAME}..."
                    withCredentials([usernamePassword(
                      credentialsId: '3f9cd7d8-c8c7-4b57-a430-2dc0753c901c',
                      usernameVariable: 'USERNAME',
                      passwordVariable: 'PASSWORD'
                    )]) {
                      sh 'echo "$USERNAME:$PASSWORD something hidden here"'
                    }
                    sh '''
                        mkdir -p "${env.TARGET_DIR}"
                        echo "fake jar content" > "${env.TARGET_DIR}"/app.jar
                    '''
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
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
                        sh '''
                        echo "Running unit tests"
                        mkdir -p target/surefire-reports
                        touch target/surefire-reports/test.xml
                        '''
                    }
                    post {
                        always {
                            junit 'target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        echo "Running integration tests"
                        sh '''
                        echo "Running integration tests"
                        mkdir -p target/failsafe-reports
                        touch target/failsafe-reports/it.xml
                        '''
                    }
                    post {
                        always {
                            junit 'target/failsafe-reports/*.xml'
                        }
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
