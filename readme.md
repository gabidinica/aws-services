In this chapter, you'll find 4 projects:
## 1. Deploy Web Application on EC2 Instance (Manually)
- Repository: [deploy-app-on-EC2-instance-manually](https://github.com/gabidinica/aws-services/tree/main/deploy-app-on-EC2-instance-manually)  
- Description: Manual deployment of a web application on an AWS EC2 instance.

## 2. CD - Deploy Application from Jenkins Pipeline to EC2 Instance (Automatically with Docker)
- Repository: [deploy-app-from-jenkins-pipeline-to-EC2-instance-with-docker](https://github.com/gabidinica/aws-services/tree/main/deploy-app-from-jenkins-pipeline-to-EC2-instance-with-docker)  
- Description: Automate deployment of applications from Jenkins pipeline using Docker.

## 3. CD - Deploy Application from Jenkins Pipeline on EC2 Instance (Automatically with Docker-Compose)
- Repository: [deploy-app-from-jenkins-with-docker-compose](https://github.com/gabidinica/aws-services/tree/main/deploy-app-from-jenkins-with-docker-compose)  
- Description: Deploy applications on EC2 using Jenkins pipeline with Docker Compose.

## 4. Complete the CI/CD Pipeline (Docker-Compose, Dynamic Versioning)
- Repository: [interacting-with-aws-cli](https://github.com/gabidinica/aws-services/tree/main/interacting-with-aws-cli)  
- Description: Full CI/CD pipeline automation including Docker Compose and dynamic versioning.

---

The link to the project used in the lecture can be found here:  
- [https://github.com/techworld-with-nana/react-nodejs-example](https://github.com/techworld-with-nana/react-nodejs-example)  
- [https://github.com/gabidinica/react-nodejs-example.git](https://github.com/gabidinica/react-nodejs-example.git)  

We need to clone locally and then build as a Docker image and push to a Docker Hub private repository.  
The app starts at port **3080** inside the Docker container. You'll need to also do a `docker login` before all of these steps.  

### Steps to build and push the image to Docker Hub:
1. Build the image:
```bash
docker build -t my-app:1.0 .
```

2. Tag the image:
```bash
docker tag my-app:1.0 private_repo:1.0
```

3. Push the image:
```bash
docker push private_repo:1.0
```
