## Demo Project

Deploy Web Application on EC2 Instance (manually)

## Technologies used

- AWS, Docker, Linux

## Project Description

- [Create and configure an EC2 Instance on AWS](#create-and-configure-an-ec2-instance-on-aws)
- [Install Docker on remote EC2 Instance](#install-docker-on-remote-ec2-instance)
- [Deploy Docker image from private Docker repository on EC2 Instance](#deploy-docker-image-from-private-docker-repository-on-ec2-instance)

---

### Create and configure an EC2 Instance on AWS

Login into AWS with the Admin account. Search for **EC2**.

1. Click on **Launch Instance**.
2. **Name**: `my-instance`
3. Click on **Add additional tag** and select:  
   - **Key**: `Type`  
   - **Value**: `web-server-with-Docker`
4. In the section **Application and OS Images (Amazon Machine Image)**:  
   - Select **Amazon Linux 2023 kernel - 6.1 AMI**  
   - For **Instance Type**: `t2.micro`
5. **Key Pair** section:  
   Click on **create a new key pair** named `docker-server`.  
   - If you are on Linux/Mac, choose `.pem` format.  
   - If you are on Windows, select `.ppk` format.  
   Then click on **Create Key pair**.
6. **Network settings** section:  
   Click on **Edit**.  
   Under **Security group** section, name the security group as: `security-group-docker-server` and in **Source Type** select **My IP**.
7. Click on **Launch Instance**.
8. Click on **EC2 Dashboard** and check for running instances. Then click on your instance.

#### Configure EC2 Instance:

1. Copy `.pem` file to ssh directory:  
```bash
mv Downloads/docker-server.pem ~/.ssh/
```

2. Verify the file exists and check permissions:
```bash
ls -l ~/.ssh/docker-server.pem
```

3. Restrict permissions on the `.pem` file:
```bash
chmod 400 ~/.ssh/docker-server.pem
```

4. Get the *public IP address* of your EC2 instance from the EC2 Dashboard.
5. Connect to the instance via SSH: 
```bash
ssh -i ~/.ssh/docker-server.pem ec2-user@INSTANCE_IP
```

6. Once connected, verify your user by typing:
```bash
whoami
```

> The output should be `ec2-user`.

### Install Docker on remote EC2 Instance

1. Update package manager tool:  
```bash
sudo yum update
```

2. Install Docker:
```bash
sudo yum install docker
```

3. Docker version: `docker --version`
4. Clear the terminal: `clear`
5. Start Docker daemon:
```bash
sudo service docker start
```

6. To check that Docker is running:
```bash
ps aux | grep docker
```

7. Add `ec2-user` to docker group:
```bash
sudo usermod -aG docker $USER
```

8. For the change to take effect, logout and login again: `exit` and then `ssh -i ~/.ssh/docker-server.pem ec2-user@INSTANCE_IP
`
9. Type `groups` and check for docker group now.

### Deploy Docker image from private Docker repository on EC2 Instance

1. Docker login and add your username and password.
2. Type: `ls -a` and check for `.docker`.
3. Docker pull the image:
```bash
docker pull repo_name:1.0
```

4. List Docker images and check for the image:
```bash
docker images
```

5. Run Docker container:
```bash
docker run -d -p 3000:3080 repo_name:1.0
```

6. Check running containers: `docker ps`
7. To access the app from browser:
- Go to AWS console
- Open *Security Groups*
- Click *Edit Inbound rules*
- Add a rule:
	- Type: *Custom TCP*
	- Port: *3000*
	- Source: *0.0.0.0/0 (Anywhere)*
- Save rules.

8. Get the public IP address of your instance, then open in browser: `http://IP_INSTANCE:3000`

