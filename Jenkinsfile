pipeline {
    agent any

    tools {
        jdk 'Jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_IMAGE = "sandydocker19/amazon-prime-app:${BUILD_NUMBER}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }

        }

        stage('Git Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/sandy193/AmazonPrime-DevSecOps-Project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon-Prime \
                        -Dsonar.projectKey=Amazon-prime '''
                    }
                }
            }
        }

       stage('Quality Gate Check') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-secret'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('OWASP Dependency Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-CHECK'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Trivy Scan For File System') {
            steps {
                sh "trivy fs --format table -o trivy-files-report.html ."
            }
        }

        stage('Build and Push Docker Image') {
          environment {
             REGISTRY_CREDENTIALS = credentials('dockerhub-secret')
            }
          steps {
            script {
              sh 'docker context use default'  
              sh "docker build -t ${DOCKER_IMAGE} ."
              def dockerImage = docker.image("${DOCKER_IMAGE}")
              docker.withRegistry('https://index.docker.io/v1/', "dockerhub-secret") {
                  dockerImage.push()
                }
            }
          }
        }

        stage('Scan Docker Image') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html ${DOCKER_IMAGE}"
            }
        }
    }     
}
