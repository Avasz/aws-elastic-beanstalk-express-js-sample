pipeline {
    agent none
    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
        DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD') // Docker Hub password stored in Jenkins credentials manager
        DOCKER_HUB_USER = "bibeki07"
        IMAGE_NAME = "assignment2"
        TAG = "initial"
    }
    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '20')) // Retain logs for 30 days / 20 builds
        timestamps() // Add timestamps to console logs
    }
    stages {
        // Install NodeJS dependencies in package.json using npm install
        stage('Install NodeJS Dependencies') {
            agent {
                docker { image 'node:16'; args '-u root:root' }
            }
            steps {
                echo "=== Installing NodeJS Dependencies ==="
                sh 'npm install --save | tee install-dependencies.log'
            }
        }
		
        // Run unit tests (if tests exist, run them, else log "no test found")
        stage('Run unit tests') {
            agent {
                docker { image 'node:16'; args '-u root:root' }
            }
            steps {
                echo "=== Running Unit Tests ==="
                sh '''
                  if [ -f package.json ] && grep -q '"test"' package.json; then
                    npm test | tee unit-tests.log
                  else
                    echo "no test found" | tee unit-tests.log
                  fi
                '''
            }
        }

        // Run security scan using snyk
        stage('Run security scan') {
            agent {
                docker { image 'node:16'; args '-u root:root' }
            }
            steps {
                echo "=== Running Security Scan with Snyk ==="
                sh 'npm install -g snyk@latest | tee snyk-install.log'
                sh 'snyk auth $SNYK_TOKEN | tee snyk-auth.log'
                sh 'snyk test --severity-threshold=high | tee snyk-test.log'
            }
        }
   
        // Build and push docker image to Docker Hub
        stage('Build Docker Image and Push to Dockerhub') {
            agent {
                docker { image 'docker:24'; args '--privileged -v /var/run/docker.sock:/var/run/docker.sock' }
            }
            steps {
                echo "=== Building Docker Image ==="
                sh 'docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$TAG . 2>&1 | tee docker-build.log'
                echo "=== Logging into Docker Hub ==="
                sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin 2>&1 | tee docker-login.log'
                echo "=== Pushing Docker Image ==="
                sh 'docker push $DOCKER_HUB_USER/$IMAGE_NAME:$TAG 2>&1 | tee docker-push.log'
            }
        }
    }
    post {
        always {
            echo "=== Build Complete: Archiving all logs ==="
            script {
                node('master') {
                    // Archive all logs from all stages
                    archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
                }
            }
        }
    }
}

