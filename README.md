# Description:
AWS_Pipeline is an advanced Flask web application that provides real-time Crypto prices while utilizing a SQL database to store historical price data. The application is deployed on an AWS environment server, showcasing a robust and scalable infrastructure.

# Key Achievements:
- Created a feature-rich Flask web app that fetches and displays real-time Crypto prices.
- Implemented a SQL database to store and retrieve historical price information, enabling users to track price trends.
- Utilized AWS for hosting the application, ensuring high availability and reliability.
- Developed an efficient Jenkins pipeline that automates the entire CI/CD workflow, enabling seamless integration and deployment.
- Employed Docker to containerize the application and enable consistent deployments across different environments.
- Utilized Docker-compose to deploy the application and database on a Test instance in AWS, allowing thorough testing before deploying to Production.
- Implemented curl checks as part of the deployment process to verify application functionality before moving to Production.
  
# Technical Highlights:
- Technology Stack: Flask, SQL, AWS (EC2), Docker, Jenkins, Docker-compose.
- Infrastructure: AWS environment server hosting the application and database.
- Version Control: Utilized Git for source code management and collaboration.
- Continuous Integration (CI): Jenkins pipeline automatically triggers builds and tests upon new code commits.
- Continuous Deployment (CD): Automated deployment of the application to the Test instance and Production server, ensuring a smooth CI/CD workflow.
- Scalability and High Availability: Leveraged AWS to design a scalable and fault-tolerant infrastructure.
- Dockerization: Containerized the application, simplifying deployment and reducing environment inconsistencies.
- Testing: Implemented curl checks during the deployment process to verify application health before moving to Production.
  
# Lessons Learned:
- Gained extensive experience in building Flask web applications and working with SQL databases.
- Mastered AWS services for application hosting and infrastructure management.
- Enhanced skills in using Docker and Docker-compose for efficient containerization and deployment.
- Developed expertise in setting up and configuring Jenkins pipelines for CI/CD automation.
- Gained a deeper understanding of DevOps principles and best practices for software development and deployment.
  
# Future Enhancements:
- Implementing user authentication and authorization to secure sensitive data and restrict access to specific features.
- Adding data visualization tools to present historical Crypto price trends in a user-friendly manner.
- Setting up monitoring and logging solutions to gain insights into application performance and identify potential issues proactively.
- Exploring automated scaling options in AWS to handle varying workloads and ensure optimal resource utilization.

# Conclusion:
AWS_Pipeline showcases a well-architected DevOps project, incorporating cutting-edge technologies and best practices. The implementation of a comprehensive CI/CD pipeline, Docker containerization, and AWS infrastructure demonstrates a commitment to delivering high-quality solutions. With valuable lessons learned and a vision for future enhancements, AWS_Pipeline highlights a strong foundation in DevOps principles and sets the stage for continued growth in this exciting field.


# Pipeline_Docker_Compose
-cloning the flask project to local jenkins machine.
-building the app as docker on the local machine, and pushing it to dockerhub.
-moving the database file, and the docker-compose.yml file to the aws Test instance.
-then starting the services with 'docker-compose up' which than pulls the image we made earlier from Docker Hub.
-testing the running flask with curl check.
-deploying to Production server.

# prerequisites: 
aws configure insatll the awscli and have a user with access credentials in aws. dockerhub, jenkins, git on local machine.
download docker and docker-compose on local machine.
role to the instances.

what I learned:
that its best to build containers outside of test and prod machines.
that i can build a container with several tags. i did one for auto versioning the images, and other to pull with to aws. (because the "$buildnumber" is a jenkins variable).
the only problem is that i have to remove all containers for the latest to deploy.
