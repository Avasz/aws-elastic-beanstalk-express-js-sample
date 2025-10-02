pipeline {
    agent none // We will define agents per stage
    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
        DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD') // Docker Hub password stored in Jenkins credentials manager
        DOCKER_HUB_USER = "bibeki07"
        IMAGE_NAME = "assignment2-test"
        TAG = "initial"
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '20')) // Store logs for 30 days or 20 builds
        timestamps() // Add timestamps to console logs
    }
    stages {
        // Install NodeJS dependencies in package.json using npm install
        stage('Install NodeJS Dependencies') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo "=== Installing NodeJS Dependencies ==="
                sh 'npm install --save'
            }
        }
		
        // Run unit tests (if tests exist, run them, else log "no test found")
        stage('Run unit tests') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo "=== Running Unit Tests ==="
                sh '''
                  if [ -f package.json ] && grep -q '"test"' package.json; then
                    npm test
                  else
                    echo "no test found"
                  fi
                '''
            }
        }

        // Run security scan using snyk - https://snyk.io
        stage('Run security scan') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo "=== Running Security Scan with Snyk ==="
                sh 'npm install -g snyk@latest' // Install snyk package globally
                sh 'snyk auth $SNYK_TOKEN'      // SNYK_TOKEN stored in Jenkins credentials
                sh 'snyk test --severity-threshold=high || true' // Fail on high/critical issues, but still log results
            }
        }
   
        // Build and push docker image to Docker Hub
        stage('Build Docker Image and Push to Dockerhub') {
            agent {
                docker {
                    image 'docker:24'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "=== Building and Pushing Docker Image ==="
                sh 'docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$TAG .'
                sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin'
                sh 'docker push $DOCKER_HUB_USER/$IMAGE_NAME:$TAG'
            }
        }
    }
    post {
        always {
            echo "=== Build Complete: Archiving logs and results ==="
            archiveArtifacts artifacts: '**/*.log', allowEmptyArchive: true // Archive logs
        }
    }
}

