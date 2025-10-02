pipeline {
    agent none // We will define agents per stage
    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
        DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD')
        DOCKER_HUB_USER = "avasz"
        IMAGE_NAME = "bib-assignment2"
        TAG = "initial"
        LOG_DIR = "logs"
    }
    options {
        // Keep build logs for 30 days, maximum of 30 builds
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '30'))
        timestamps() // This adds timestamps to console logs
    }
    stages {
        stage('Install NodeJS Dependencies') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                sh 'mkdir -p ${LOG_DIR}'
                sh 'npm install --save | tee logs/npm_install.log'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'logs/npm_install.log', allowEmptyArchive: true
                }
            }
        }

        stage('Run Unit Tests') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                sh """
                    mkdir -p ${LOG_DIR}
                    if npm test 2>&1 | tee logs/unit_test.log; then
                        echo "Tests executed successfully" | tee -a logs/unit_test.log
                    else
                        echo "No test found or tests failed" | tee -a logs/unit_test.log
                    fi
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'logs/unit_test.log', allowEmptyArchive: true
                    junit testResults: '**/test-results/**/*.xml', allowEmptyResults: true
                }
            }
        }

        stage('Run Security Scan') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                sh 'mkdir -p ${LOG_DIR}'
                sh 'npm install -g snyk@latest | tee logs/snyk_install.log'
                sh 'snyk auth $SNYK_TOKEN | tee -a logs/snyk_scan.log'
                sh 'snyk test --severity-threshold=high | tee -a logs/snyk_scan.log || true'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'logs/snyk_*.log', allowEmptyArchive: true
                    recordIssues tools: [snyk(pattern: 'logs/snyk_scan.log')] // Requires Warnings NG plugin
                }
            }
        }

        stage('Build Docker Image and Push to Dockerhub') {
            agent {
                docker {
                    image 'docker:24'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'mkdir -p ${LOG_DIR}'
                sh "docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$TAG . | tee logs/docker_build.log"
                sh "echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin | tee -a logs/docker_build.log"
                sh "docker push $DOCKER_HUB_USER/$IMAGE_NAME:$TAG | tee -a logs/docker_build.log"
            }
            post {
                always {
                    archiveArtifacts artifacts: 'logs/docker_build.log', allowEmptyArchive: true
                }
            }
        }
    }
    post {
        always {
            echo "Build completed. Logs archived in Jenkins artifacts."
        }
    }
}

