# Enable docker in jenkins

Enable of docker in jenkins can be done in two ways based on how you configured the jenkins that is 
  - if you configured jenkins via docker image then it can be done in different way
  - same goes for the other way like if you download jenkins directly on the os it can be done differently.

First if you are using jenkins via a docker iamge then we need not to be install docker again inside the container to have access of docker commands inside the container, we can use the concept of docker volumes

  - like if you are using jenkins via docker image you already used docker volumes when mounting the jenkins data to the server
  - ```bash
    docker run -p 8080:8080 -p 50000:50000 -d \
    -v jenkins_home:/var/jenkins_home  jenkins/jenkins:lts
    # here you used the volumes for storing the jenkins data
    # and if you observe we alreday have docker in our host server so the docker files will be available in our host server so we can also make them available inside the conatiner by mounting them
    #the modified command will be
    docker run -p 8080:8080 -p 50000:50000 -d \
    -v jenkins_home:/var/jenkins_home \
    -v /var/run/docker.sock:/var/run/docker/sock jenkins/jenkins:lts #docker.sock file is a unix socket file, used by docker deamon to communicate to docker client

    ```
    if you don't understand this you can go to my other repo [installing jenkins on server](https://github.com/Hemanth42d/install-jenkins-on-server)

    - After completing the above steps check if docker is available inside the container or not before that we need to give premissions for the all the users to run the docker commands
    - ```bash
      # by default if we enter into the container we will be in the jenkins user but this time we need to login as root user so run the below command
      docker exec -u 0 -it <container_id> /bash/sh

      # to fix everything and ensure smooth running of docker inside container we need to run the below command
      curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall
      
      ls -l /var/run/docker.sock
      chmod 666 /var/run/docker.sock # remember that we loged in as root so need to execute it with sudo
      # 666 basically means enable the read and write permissions for every one for that particular file

      #then login as jenkins user
      docker exec -it <container_id> /bash/sh
      docker pull hello-world # to check whether docker is running or not
      ```

## Build Docker image from jenkins

Create a job for example here i am going for an java maven app

[link for the demo java maven app](https://github.com/Hemanth42d/java-maven-app-learning-jenkins.git)

1. After creating a job go for > configuration in the side bar and then for > Build steps
     > configuration > Build Steps
2. First Build step will be using maven and we already have a pulgin for maven and the using dropdown click on invoke top-level maven target
   - and for maven version give any of the versions you have
   - and for goal give the command **package** for **building and testing the application**
3. Next use Execute shell from the dropdown for executing docker commands
   - ```bash
     docker build -t java-maven-app:1.0 .
     ```
     - save the configuration and then in sidebar click on Build now.
4. Ater build successfull you can check it in the jenkins user by running the command
   - ```bash
     docker images
     ```
   - you can see the java-maven-app image in the output


## Push Image to the Docker Repository

1. First step will be creating the credentials for it to push the docker image to repository.
   - Create Docker hub account
   - Configur Credential in jenkins.
  
2. Go to Docker hub and create a private repository
3. Then Go to Jenkins Dashboar
   > Manage Jenkins > Credentials > System > Gobal Credentials >
   > Give Your username and password of Docker hub in the respective fields

4. So pushing of the docker image has already seen in one of my repo [check it out](https://checkout.com/)
    ```bash
    docker build -t java-maven-app:1.0 . (build image with the repo name so we don't need to explicitly mention again while pushing)
    # we can get the credentials via a plugin in our jenkins named use secret files Or text (in the Build Environment) > click on add > we get a dropdown > click on > username and password > give variable names as USERNAME and PASSWORD and choose the credentials
    docker login -u $USERNAME -p $PASSWORD # docker hub will be the default for login and if you have your nexus or aws ecr repo's you need to specify it here.
    docker push java-maven-app:1.0
    ```
    - save and build it.
    - The best practice in docker while giving the password credential is by providing it in standard input format
      ```bash
      docker echo $PASSWORD | docker login -u $USERNAME --password-stdin
      ```

  5. While learning it and if we are running it on the https then we need to allow docker to push to the insecure registries and do it in the deamon.json, By default the file won't be their.
     ```bash
     vim /etc/docker/deamon.json (in the server terminal not in the docker container if your are running jenkins as a container)

     #file { "insecure-registries":["<ip_address>:<port_number>"]}
     # enable the permissions again for the /var/run/docker.sock file since the jenkins container is restarted if using jenkins as docker container.

6. first thing you should do now is to create nexus credentials in the jenkins.
   > Then Go to Jenkins Dashboar
   > Manage Jenkins > Credentials > System > Gobal Credentials >
   > Give Your username and password of nexus in the respective fields
7. Instead of docker hub credentials choose nexus credentials.
8. Push the image
   ```bash
    docker build -t <ip_address>:<port_number>/java-maven-app:1.0 . (build image with the repo name so we don't need to explicitly mention again while pushing)
    # we can get the credentials via a plugin in our jenkins named use secret files Or text (in the Build Environment) > click on add > we get a dropdown > click on > username and password > give variable names as USERNAME and PASSWORD and choose the credentials
    # docker hub will be the default for login and if you have your nexus or aws ecr repo's you need to specify it here.
    docker login -u $USERNAME -p $PASSWORD ip_address>:<port_number>
    docker push <ip_address>:<port_number>/java-maven-app:1.0
    ```








    
