# Demo Project:  
**CD - Deploy Application from Jenkins Pipeline to EC2 Instance**  
*(automatically with docker)*  

## Technologies used:
AWS, Jenkins, Docker, Linux, Git, Java, Maven, Docker Hub  

## Project Description:
- [Prepare AWS EC2 Instance for deployment (Install Docker)](#prepare-aws-ec2-instance-for-deployment-install-docker)  
- [Create ssh key credentials for EC2 server on Jenkins](#create-ssh-key-credentials-for-ec2-server-on-jenkins)  
- [Extend the previous CI pipeline with deploy step to ssh into the remote EC2 instance and deploy newly built image from Jenkins server](#extend-the-previous-ci-pipeline-with-deploy-step-to-ssh-into-the-remote-ec2-instance-and-deploy-newly-built-image-from-jenkins-server)  
- [Configure security group on EC2 Instance to allow access to our web application](#configure-security-group-on-ec2-instance-to-allow-access-to-our-web-application)  

---

### Prepare AWS EC2 Instance for deployment (Install Docker)

1. Update package manager tool:  
```bash
sudo yum update
```

2. Install Docker:
```bash
sudo yum install docker
```

3. Check Docker version: `docker --version`
4. Clear terminal: `clear`
5. Start Docker daemon:
```bash
sudo service docker start
```

6. Check that Docker is running:
```bash
ps aux | grep docker
```

7. Add `ec2-user` to Docker group:
```bash
sudo usermod -aG docker $USER
```

8. Logout for changes to take effect: `exit`
9. Connect again to instance: `ssh -i ~/.ssh/docker-server.pem ec2-user@INSTANCE_IP`
10. Check groups to verify Docker group: `groups`

### Create ssh key credentials for EC2 server on Jenkins

1. Copy your droplet IP address and paste it into a new browser page: `DROPLET_IP:8080`
2. In Jenkins, go to **Manage Jenkins → Plugins**, search for **SSH Agent**, and install it.
3. In terminal, type:  
```bash
ssh -i .ssh/docker-server.pem ec2-user@EC2_IP
```

4. In Jenkins, add a new **aws-multibranch-pipeline** and connect it to the **java-maven-app** repository:  
[https://github.com/gabidinica/java-maven-app](https://github.com/gabidinica/java-maven-app)
5. Click on the pipeline created, then go to **Credentials**.  
Under *Stores scoped to aws-multibranch-pipeline*, click on the store name.
6. Click on **Global credentials (unrestricted)**, then on **Add credentials**.
7. Fill in the fields:

- **Kind:** SSH Username with private key
- **ID:** `ec2-server-key`
- **Username:** `ec2-user`
- **Private Key:** Copy the content of `docker-server.pem`
 
8. In terminal, type:
`cat .ssh/docker-server.pem` and copy the output and paste it into the **Private Key** field under *Enter directly*.
9. Click Create.

### Extend the previous CI pipeline with deploy step to ssh into the remote EC2 instance and deploy newly built image from Jenkins server

1. Open the project in **Visual Studio Code** and update the Jenkinsfile to include the SSH agent.  

In the **Deploy** step, add:  

```bash
sshagent(['ec2-server-key']) {
    def dockerCmd = 'docker run -p 3080:3080 -d dockerhub-repo:1.0'
    sh "ssh -o StrictHostKeyChecking=no ec2-user@INSTANCE_IP ${dockerCmd}"
}
```

> Suppress the SSH pop-up is: -o StrictHostKeyChecking=no
> dockerhub-repo:1.0- replace with your docker hub repository and 1.0 is the version from the repo of the project

2. Commit and push the changes to GitHub.

3. In terminal, log in to the EC2 instance and check that there are no images and containers:  
```bash
docker images
docker ps
```

### Configure security group on EC2 Instance to allow access to our web application

In AWS Console, go to **EC2 instance**, click on **Security**, and then click on the **security group** attached.  

1. **Edit Inbound rules** and **Add rule**:  
   - **Type:** SSH  
   - **Source:** Add the IP address of your Jenkins server  
   - Click **Save Rules**

2. For the first IP, change the **Port range** to `3080`, where the app will be accessible from the browser.
3. In Jenkins, in the **multibranch pipeline** on the `feature/payment` branch where we updated the build, start the build.
4. Check the logs.
5. In terminal, where you’re logged into the EC2 instance, check again with:  
```bash
docker images
docker ps
```

6. In a browser, enter the **public IP** of the instance and port `80`, then check that the app is working.

