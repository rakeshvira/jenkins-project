pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE_NAME', description: 'Enter the Docker image name', defaultValue: 'shivanandareddy452/custom-img-17-04-2024')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_id')
    }

    stages {
        stage('Update Yum') {
            steps {
                script {
                    sh 'sudo yum update -y'
                }
            }
        }

        stage('Force Cleanup') {
            steps {
                script {
                    // Stop and remove all running containers
                    sh 'sudo docker stop $(sudo docker ps -a -q) || true'
                    sh 'sudo docker rm $(sudo docker ps -a -q) || true'

                    // Remove all Docker images
                    sh 'sudo docker rmi $(sudo docker images -q) || true'
                }
            }
        }

        stage('Install Git') {
            steps {
                script {
                    sh 'sudo yum install -y git'
                }
            }
        }

        stage('Install Docker') {
            steps {
                script {
                    sh 'sudo yum install -y docker'
                    sh 'sudo systemctl restart docker'
                    sh 'sudo systemctl enable docker'
                }
            }
        }

        stage('Install Docker Compose') {
            steps {
                script {
                    sh 'sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose'
                    sh 'sudo chmod +x /usr/local/bin/docker-compose'
                    sh 'docker-compose version'
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    sh 'sudo yum install -y git'
                    sh 'rm -rf /tmp/jenkins-cicd-docker'
                    sh 'git clone https://github.com/awscloudrockers/jenkins-cicd-docker.git /tmp/jenkins-cicd-docker'
                    sh 'ls /tmp/jenkins-cicd-docker'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "sudo docker build -t ${params.DOCKER_IMAGE_NAME} /tmp/jenkins-cicd-docker/"
                }
            }
        }

        stage('Show Docker Images') {
            steps {
                script {
                    sh 'sudo docker images'
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh "sudo docker run -dit --name custom-container ${params.DOCKER_IMAGE_NAME}"
                }
            }
        }

        stage('Show Docker Containers') {
            steps {
                script {
                    sh 'sudo docker ps -a'
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'dockerhub_id', variable: 'DOCKERHUB_CREDENTIALS_PSW')]) {
                        sh "echo \${DOCKERHUB_CREDENTIALS_PSW} | sudo docker login -u shivanandareddy452 --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    sh "sudo docker push ${params.DOCKER_IMAGE_NAME}"
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image build, run, and push successful!'
        }
        failure {
            echo 'Docker image build, run, or push failed.'
        }
    }
}
