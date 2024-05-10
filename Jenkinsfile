pipeline {
    agent any
    parameters {
        string(name: 'DOCKER_TAG', description: 'Enter the tag for the Docker image', defaultValue: 'latest')
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository containing the Dockerfile
                git branch: 'main',
                  url: 'https://github.com/gaman5575/docker-jenkins-pipleine.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Make sure Docker is installed and configured properly on Jenkins
                    def dockerImage = docker.build("docker.io/gaman5575/python-jenkins-app:${params.DOCKER_TAG}", "-f Dockerfile .")
                 //   def dockerCommand = "/usr/bin/docker" // Update the path to the docker command
                 //   def dockerImage = "${dockerCommand} build -t docker.io/gaman5575/python-jenkins-app:${params.DOCKER_TAG} -f Dockerfile ."
                    docker.withRegistry('', 'dockerhub-security') {
                        dockerImage.push("${params.DOCKER_TAG}")
                    }
                    // You can add any additional build arguments if needed
                }
            }
        }

        stage('Scan Docker Image for Vulnerabilities') {
            steps {
                sh "docker run -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.51.1 image docker.io/gaman5575/python-jenkins-app:${params.DOCKER_TAG}"
            }
        }

        stage('Update Image Tag in deployment.yaml') {
            steps {
                script {
                    // Replace the image tag in deployment.yaml with the specified DOCKER_TAG
                    sh "sed -i 's|image:.*|image: docker.io/gaman5575/python-jenkins-app:${params.DOCKER_TAG}|g' deployment.yaml"
                }
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    // Retrieve kubeconfig secret from Jenkins credentials
                    withKubeConfig([credentialsId: 'k8s-config', serverUrl: 'https://10.0.0.100:6443']) {
                        // Authenticate with Kubernetes cluster
                        sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
}
