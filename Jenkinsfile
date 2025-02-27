pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SQ'  // Name of your SonarQube server in Jenkins
        DOCKER_IMAGE = 'helloworld-app:latest' // Replace with your desired image name
        IMAGE_TAG = 'latest'
        IMAGE_NAME = 'helloworld-app' // Image name
        DOCKERHUB_USERNAME = 'avneetkour'  // Replace with your DockerHub username
        DOCKERHUB_REPO = "${DOCKERHUB_USERNAME}/${IMAGE_NAME}" // DockerHub repository
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials1' // Jenkins DockerHub credentials ID
        DOCKER_REGISTRY = 'docker.io' 
    }

    stages {
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage(' Docker Run') {
            steps {
                sh 'docker run -d -p 5000:5000 $DOCKER_IMAGE '
            }
        }
         stage('Scan Docker Image with Trivy') {
             steps {
                sh 'trivy image $DOCKER_IMAGE'
            }
         }

        stage('Authenticate with DockerHub') {
            steps {
                // Authenticate with DockerHub
                withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                     sh '''
                            echo $DOCKER_PASSWORD | docker login $DOCKER_REGISTRY -u $DOCKER_USERNAME --password-stdin
                        '''
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    // Tag the Docker image for DockerHub
                    sh "docker tag $DOCKER_IMAGE $DOCKERHUB_REPO:$IMAGE_TAG"
                    
                    // Push the image to DockerHub
                    sh "docker push $DOCKERHUB_REPO:$IMAGE_TAG"
                    
                    // Confirm the image is in DockerHub
                    sh 'docker images'
                }
            }
        }
    }
}


