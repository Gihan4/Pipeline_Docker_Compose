pipeline {
    agent any

    environment {
        testip = sh(
            script: "aws ec2 describe-instances --region eu-central-1 --filters 'Name=tag:Environment,Values=Compose1' --query 'Reservations[].Instances[].PublicIpAddress' --output text",
            returnStdout: true
        ).trim()
        prodip = sh(
            script: "aws ec2 describe-instances --region eu-central-1 --filters 'Name=tag:Environment,Values=P1' --query 'Reservations[].Instances[].PublicIpAddress' --output text",
            returnStdout: true
        ).trim()
    }

    stages {
        stage('Cleanup') {
            steps {
                echo "Cleaning up..."
                // removes all files and directories in the current working directory 
                sh 'rm -rf *'
            }
        }

        stage('Install Docker and Docker-compose on Instances') {
            steps {
              
                echo "Installing Docker and Docker Compose on AWS test instance..."
                sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/Gihan4.pem ec2-user@${testip} '
                        sudo yum update -y &&
                        sudo yum install -y docker &&
                        sudo service docker start &&
                        sudo usermod -aG docker ec2-user &&
                        sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose &&
                        sudo chmod +x /usr/local/bin/docker-compose'
                '''       

                echo "Installing Docker and Docker Compose on AWS production instance..."
                sh '''
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/Gihan4.pem ec2-user@${prodip} '
                        sudo yum update -y &&
                        sudo yum install -y docker &&
                        sudo service docker start &&
                        sudo usermod -aG docker ec2-user &&
                        sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose &&
                        sudo chmod +x /usr/local/bin/docker-compose'
                '''  
                
            }
        }

        stage('Stop and Remove Containers and Images from all machines') {
            steps {
                // Delete from Jenkins server
                echo "Stopping and removing containers and images on Jenkins server..."
                sh "docker stop \$(docker ps -aq) || true"
                sh "docker rm \$(docker ps -aq) || true"
                sh "docker rmi -f \$(docker images -aq) || true"
        
                // Delete from AWS Test instance
                echo "Stopping and removing containers and images on AWS Test instance..."
                sshCommand remote: ec2TestInstance, command: '''
                    docker stop \$(docker ps -aq) || true
                    docker rm \$(docker ps -aq) || true
                    docker rmi -f \$(docker images -aq) || true
                '''
        
                // Delete from AWS Production instance
                echo "Stopping and removing containers and images on AWS Production instance..."
                sshCommand remote: ec2ProdInstance, command: '''
                    docker stop \$(docker ps -aq) || true
                    docker rm \$(docker ps -aq) || true
                    docker rmi -f \$(docker images -aq) || true
                '''
            }
        }



        stage('Clone') {
            steps {
                echo "Cloning repository..."
                sh 'git clone https://github.com/Gihan4/docker_compose_project.git'
                sh 'ls'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image from app..."
                dir('docker_compose_project') {
                    sh 'docker build -t gihan4/appimage:${BUILD_NUMBER} -t gihan4/appimage:latest -f app/Dockerfile .'
                }
            }
        }

        stage('Push app Image to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                sh 'docker push --all-tags gihan4/appimage'
            }
        }

        stage('Deploy on Test server') {
            steps {
                echo "Deploying and testing on AWS test instance..."
                    // Copy the docker-compose.yml file to the EC2 instance.
                    sh "scp -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem /var/lib/jenkins/workspace/PIpeline_compose/docker_compose_project/docker-compose.yml ec2-user@${testip}:~/docker-compose.yml"
                    // Copy the database folder to the EC2 instance.
                    sh "scp -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem -r /var/lib/jenkins/workspace/PIpeline_compose/docker_compose_project/database ec2-user@${testip}:~/"
                    // SSH into the EC2 instance and run docker-compose that will pull the app image from Docker Hub.
                    sh "ssh -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem ec2-user@${testip} 'docker-compose -f ~/docker-compose.yml up -d'"
            }
        }

        stage('Check Flask API') {
            steps {
                echo "Checking Flask API..."
    
                // Make an HTTP request to the Flask API endpoint
                script {
                    def response = sh script: "curl -s -o /dev/null -w '%{http_code}' http://${testip}:5000", returnStdout: true
                    def statusCode = response.trim()
                    
                    if (statusCode == '200') {
                        echo "Flask API is running successfully. Response code: ${statusCode}"
                    } else {
                        error "Flask API is not running. Response code: ${statusCode}"
                    }
                }
            }
        }


        stage('Deploy on Production server') {
            steps {
                echo "Deploying after testing on AWS production instance..."
                    // Copy the docker-compose.yml file to the EC2 instance.
                    sh "scp -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem /var/lib/jenkins/workspace/PIpeline_compose/docker_compose_project/docker-compose.yml ec2-user@${prodip}:~/docker-compose.yml"
                    // Copy the database folder to the EC2 instance.
                    sh "scp -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem -r /var/lib/jenkins/workspace/PIpeline_compose/docker_compose_project/database ec2-user@${prodip}:~/"
                    // SSH into the EC2 instance and run docker-compose that will pull the app image from Docker Hub and run the services.
                    sh "ssh -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem ec2-user@${prodip} 'docker-compose -f ~/docker-compose.yml up -d'"
            }
        }


    }
}
