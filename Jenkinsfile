pipeline {
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
        }
    }
}