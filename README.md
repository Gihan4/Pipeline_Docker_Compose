# Pipeline_Docker_Compose
cloning the flask project to local jenkins machine.
building the app as docker on the local machine, and pushing it to dockerhub.
moving the database file, and the docker-compose.yml file to the aws Test instance.
then starting the services with 'docker-compose up' which than pulls the image we made earlier from Docker Hub.

prerequisites: 
configuring aws, dockerhub, jenkins, git on local machine.
download docker and docker-compose on local machine.
role to the instances.

what I learned:
that its best to build containers outside of test and prod machines.
that i can build a container with several tags. i did one for auto versioning the images, and other to pull with to aws. (because the "$buildnumber" is a jenkins variable).

