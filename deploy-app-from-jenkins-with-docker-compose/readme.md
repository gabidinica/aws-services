# Demo Project:  
**CD - Deploy Application from Jenkins Pipeline on EC2 Instance**  
*(automatically with docker-compose)*  

#### Here is the repo of the project that we'll use:  
[https://github.com/gabidinica/java-maven-app](https://github.com/gabidinica/java-maven-app)

## Technologies used:
AWS, Jenkins, Docker, Linux, Git, Java, Maven, Docker Hub  

## Project Description:
- [Install Docker Compose on AWS EC2 Instance](#install-docker-compose-on-aws-ec2-instance)  
- [Create docker-compose.yml file that deploys our web application image](#create-docker-composeyml-file-that-deploys-our-web-application-image)  
- [Configure Jenkins pipeline to deploy newly built image using Docker Compose on EC2 server](#configure-jenkins-pipeline-to-deploy-newly-built-image-using-docker-compose-on-ec2-server)  
- [Improvement: Extract multiple Linux commands into a shell script and execute it from Jenkinsfile](#improvement-extract-multiple-linux-commands-into-a-shell-script-and-execute-it-from-jenkinsfile)  

---

### Install Docker Compose on AWS EC2 Instance

1. Update package manager tool:  
```bash
sudo yum update
```

2. Install Docker:
```bash
sudo yum install docker
```

3. Check Docker version: `docker --version`
4. Start Docker daemon:
```bash
sudo service docker start
```

5. Check that Docker is running: `ps aux | grep docker`
6. Add *ec2-user* to Docker group: 
```bash
sudo usermod -aG docker $USER
```

7. Logout for changes to take effect: `exit`
8. Connect again to instance: 
```bash
ssh -i ~/.ssh/docker-server.pem ec2-user@INSTANCE_IP
```

9. Check groups to verify Docker group: `groups`

### Create docker-compose.yml file that deploys our web application image

In terminal on EC2 instance, to install Docker Compose go to this URL:  
[https://gist.github.com/npearce/6f3c7826c7499587f00957fee62f8ee9](https://gist.github.com/npearce/6f3c7826c7499587f00957fee62f8ee9)

1. **Docker Compose curl command:**  
```bash
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

2. Fix permissions after download: 
```bash
docker-compose --version
```

3. Verify success: `docker-compose --version`
4. Check for running containers: `docker ps`
5. Remove running containers: `docker rm container_id` and remove images 'docker rmi image_id'
6. Clone locally the project **java-maven-app** and checkout a new branch, for example: `new-jenkins-jobs`.  

7. Add a new file: **docker-compose.yaml**  
```yaml
   version: '3.8'
   services:
     java-maven-app:
       image: repo_name:java-maven-1.0
       ports: 
         - 8080:8080

     postgres:
       image: postgres:15
       ports:
         - 5432:5432
       environment:
         - POSTGRES_PASSWORD=my-pwd
```

### Configure Jenkins pipeline to deploy newly built image using Docker Compose on EC2 server

We need **docker-compose.yaml** on the EC2 instance.  
Using the **Jenkinsfile**, we will copy it from the repo to the EC2 instance.

In the **Jenkinsfile**, under the `deploy` stage:  
```groovy
def dockerComposeCmd = "docker-compose -f docker-compose.yaml up --detach"

sshagent(['eco-server-key']) {
    sh "scp docker-compose.yaml ec2-user@INSTANCE_IP:/home/ec2-user"
    sh "ssh -o StrictHostKeyChecking=no ec2-user@INSTANCE_IP ${dockerComposeCmd}"
}
```

Steps:

1. Add and push the changes to the branch.
2. Go to Jenkins in the browser and start building the job.
3. Check the logs.
4. Go to the EC2 instance in the terminal and check for running containers:
`docker ps`
5. List files and check that `docker-compose.yaml` is displayed: `ls`

### Improvement: Extract multiple Linux commands into a shell script and execute it from Jenkinsfile

Add a new file in **Visual Studio Code** named: `server-cmds.sh`  

```bash
#!/usr/bin/env bash
docker-compose -f docker-compose.yaml up --detach
echo "success"
```

In **Jenkinsfile**:  

```groovy
def shellCmd = "bash ./server-cmds.sh"

sshagent(['eco-server-key']) {
    sh "scp server-cmds.sh ec2-user@INSTANCE_IP:/home/ec2-user"
    sh "scp docker-compose.yaml ec2-user@INSTANCE_IP:/home/ec2-user"
    sh "ssh -o StrictHostKeyChecking=no ec2-user@INSTANCE_IP ${shellCmd}"
}
```


1. Commit and push the changes to the repo and check the branch you’re on.
2. **Stop the containers** that are running on the EC2 instance:  
```bash
docker-compose -f docker-compose.yaml down
```

3. In Jenkins, build the job again and check the logs.
4. In the EC2 instance, check running containers:
`docker ps`
5. Replace Docker image with newly built version:
In `docker-compose.yaml` for `java-maven-app`, replace: `image: ${IMAGE}`
6. In `server-cmds.sh`, above the `docker-compose` command, add:
`export IMAGE=$1`
7. In **Jenkinsfile**, update:  
```groovy
def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
```

8. Update and push the changes on the branch.
9. Go to Jenkins and build the job again.
