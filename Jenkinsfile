pipeline {
    //agent any means that the pipeline can run on any available agent.
    agent any

    options {
        ansiColor('xterm')
    }

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:22-alpine'
                }
            }
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('test') {
            parallel {
                stage('unit tests') {
                    agent {
                        docker {
                            image 'node:22-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        // Install dependencies in the test container before running Vitest
                        sh 'npm ci'
                        sh 'npm run test:unit -- --reporter=verbose'
                    }
                }
                stage('integration tests') {
                    agent {
                        docker {
                            //use playwright docker image with 1.54.2 version and jammy base image
                            image 'mcr.microsoft.com/playwright:v1.54.2-jammy'       
                            //reuse the same node workspace to avoid re-downloading dependencies and no need to use npm ci again
                            reuseNode true                                           
                        }
                    }
                    steps {
                        sh 'npm ci'
                        sh 'npx playwright test'
                    }
                }
            }
        }

        stage('deploy') {
            agent {
                docker {
                    image 'alpine'
                }
            }
            steps {
                // Mock deployment which does nothing
                echo 'Mock deployment was successful!'
            }
        }

        stage('e2e') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
                    reuseNode true
                }
            }
            environment {
                E2E_BASE_URL = 'https://spanish-cards.netlify.app/'
            }
            steps {
                sh 'npx playwright test'
            }

            // post means that after the stage is completed, the following actions will be executed.
            post { 
                //always means that the following actions will be executed regardless of the stage result (success, failure, or unstable).                         
                always {
                    // Publish the Playwright report to Jenkins
                    publishHTML([
                        // allowMissing: false means that if the report is missing, the build will fail. 
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: false,
                        reportDir: 'reports-e2e/html/',
                        reportFiles: 'index.html',
                        reportName: 'Playwright HTML Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                    junit stdioRetention: 'ALL', testResults: 'reports-e2e/junit.xml'
                }
            }
        }
    }
}