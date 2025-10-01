pipeline {
    agent none // We will define agents per stage
    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
		DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD')
        DOCKER_HUB_USER = "avasz"
        IMAGE_NAME = "bib-assignment2"
        TAG = "initial"
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
                sh 'npm install --save'
            }
        }
        stage('Run unit tests') {
             agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                sh 'echo "no test found"'
            }
        }
        stage('Run security scan') {
             agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                sh 'npm install -g snyk@latest'
                sh 'snyk auth $SNYK_TOKEN'
                sh 'snyk test --severity-threshold=high'
           }
        }
   
        stage('Build Docker Image') {
             agent {
                docker {
                    image 'docker:24'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$TAG .'
            }
        }
        stage('Docker login and push image') {
            steps {
			script {
			// Login using stored credentials
			sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_HUB_USER --password-stdin'
			sh 'docker push $DOCKER_HUB_USER/$IMAGE_NAME:$TAG'
                }

            }
        }
    }


