In this chapter, you'll find 4 projects:
- Deploy Web Application on EC2 Instance (manually)
- CD - Deploy Application from Jenkins Pipeline to EC2 Instance (automatically with docker)
- CD - Deploy Application from Jenkins Pipeline on EC2 Instance (automatically with docker-compose)
- Complete the CI/CD Pipeline (Docker-Compose, Dynamic versioning)

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
