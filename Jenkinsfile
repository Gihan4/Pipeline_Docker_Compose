pipeline {
    agent any

    //triggers {
        // The pipeline is triggered every minute to check for changes in the git
        // pollSCM('*/1 * * * *')
    //}

    environment {
        testip = sh(
            script: "aws ec2 describe-instances --region eu-central-1 --filters 'Name=tag:Environment,Values=Compose1' --query 'Reservations[].Instances[].PublicIpAddress' --output text",
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

        stage('Stop and Remove Containers and Images') {
            steps {
                // Delete from Jenkins server
                echo "Stopping and removing containers and images on Jenkins server..."
                sh "docker stop \$(docker ps -aq) || true"
                sh "docker rm \$(docker ps -aq) || true"
                // except for the latest version of the image
                sh """
                    docker images --format '{{.Repository}}:{{.Tag}}' gihan4/myimage:* |
                    awk -F: '{print \$2}' |
                    sort -r |
                    tail -n +2 |
                    xargs -I {} docker rmi gihan4/myimage:{} || true
                """
        
                // Delete from AWS Test instance
                echo "Stopping and removing containers and images on AWS Test instance..."
                sh """
                    ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/Gihan4.pem ec2-user@${testip} '
                        docker stop \$(docker ps -aq) || true &&
                        docker rm \$(docker ps -aq) || true &&
                        docker images --format "{{.Repository}}:{{.Tag}}" gihan4/myimage:* |
                        awk -F: "{print \\\$2}" |
                        sort -r |
                        tail -n +2 |
                        xargs -I {} docker rmi gihan4/myimage:{} || true'
                """
            }
        }



        stage('Install Docker and Docker-compose on AWS Instance') {
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
                    sh 'docker build -t gihan4/appimage:${BUILD_NUMBER} -f app/Dockerfile .'
                }
            }
        }

        stage('Push app Image to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                sh 'docker push gihan4/appimage:${BUILD_NUMBER}'
            }
        }

        stage('Deploy on Test server') {
            steps {
                echo "Deploying and testing on AWS test instance..."
                    // pulls the Docker app image onto the EC2 instance.
                    sh "ssh -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem ec2-user@${testip} 'docker pull gihan4/appimage:${BUILD_NUMBER}'"
                    // Copy the docker-compose.yml file to the EC2 instance.
                    sh "scp -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem Pipeline_Docker_Compose/docker_compose_project/docker-compose.yml ec2-user@${testip}:~/docker-compose.yml"
                    // Copy the database folder to the EC2 instance.
                    sh "scp -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem -r Pipeline_Docker_Compose/docker_compose_project/database ec2-user@${testip}:~/"
                    // SSH into the EC2 instance and run docker-compose.
                    sh "ssh -o StrictHostKeyChecking=no -i $HOME/.ssh/Gihan4.pem ec2-user@${testip} 'docker-compose -f ~/docker-compose.yml up -d'"
            }
        }

        stage('Check Flask API') {
            steps {
                echo "Checking Flask API..."
        
                // Make an HTTP request to the /get_prices route
                script {
                    def response = sh script: "curl -s -o /dev/null -w '%{http_code}' http://${testip}:5000/get_prices", returnStdout: true
                    def statusCode = response.trim()
        
                    if (statusCode == '200') {
                        echo "Flask API /get_prices route is working. Response code: ${statusCode}"
                    } else {
                        error "Flask API /get_prices route is not working. Response code: ${statusCode}"
                    }
                }
            }
        }


    }
}
