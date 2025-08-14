# Demo Project:  
**Complete the CI/CD Pipeline (Docker-Compose, Dynamic versioning)**  

#### Project used here can be found: [https://github.com/gabidinica/java-maven-app](https://github.com/gabidinica/java-maven-app)


## Technologies used:
AWS, Jenkins, Docker, Linux, Git, Java, Maven, Docker Hub  

## Project Description:
- [CI step: Increment version](#ci-step-increment-version)  
- [CI step: Build artifact for Java Maven application](#ci-step-build-artifact-for-java-maven-application)  
- [CI step: Build and push Docker image to Docker Hub](#ci-step-build-and-push-docker-image-to-docker-hub)  
- [CD step: Deploy new application version with Docker Compose](#cd-step-deploy-new-application-version-with-docker-compose)  
- [CD step: Commit the version update](#cd-step-commit-the-version-update)  

---

### CI step: Increment version

In **Jenkinsfile**, add the following step:  

```groovy
stage('increment version') {
    steps {
        script {
            echo 'incrementing app version ...'
            sh 'mvn build-helper:parse-version versions:set \
            -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} \
            versions:commit'
            def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
            def version = matcher[0][1]
            env.IMAGE_NAME = "$version-$BUILD_NUMBER"
        }
    }
}
```

### CI step: Build artifact for Java Maven application

In **Jenkinsfile**, add the following step:
```groovy
stage("build app") {
    steps {
        script {
            echo 'building the application...'
            sh 'mvn clean package'
        }
    }
}
```

### CI step: Build and push Docker image to Docker Hub

Add the following step:

```groovy
  stage("build image") {
            steps {
                script {
                      echo "building the docker image..."
                       withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                       sh "docker build -t gabiand/demo-app:${IMAGE_NAME} ."
                       sh 'echo $PASS | docker login -u $USER --password-stdin'
                       sh "docker push gabiand/demo-app:${IMAGE_NAME}"
                }
            }
        }
        }
```

### CD step: Deploy new application version with Docker Compose

In **Jenkinsfile**, in the `deploy` stage:  

```groovy
def dockerComposeCmd = "docker-compose -f docker-compose.yaml up --detach"

sshagent(['eco-server-key']) {
    sh "scp docker-compose.yaml ec2-user@INSTANCE_IP:/home/ec2-user"
    sh "ssh -o StrictHostKeyChecking=no ec2-user@INSTANCE_IP ${dockerComposeCmd}"
}
```

### CD step: Commit the version update

```groovy
stage("commit version update") {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: '0e1b9c50-4389-48d2-bc25-7efeb014364b', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                sh 'git config --global user.email "jenkins@example.com"'
                sh 'git config --global user.name "jenkins"'
                sh 'git status'
                sh 'git branch'
                sh 'git config --list'
                sh "git remote set-url origin https://${USER}:${PASS}@github.com/gabidinica/java-maven-app.git"
                sh 'git add .'
                sh 'git commit -m "ci: version bump "'
                echo "⬇️ Pulling latest changes with rebase..."
                sh 'git pull --rebase origin jenkins-branch'
                sh 'git push origin HEAD:jenkins-branch'
            }
        }
    }
}
```

> Instead of my url for github, use yours where the java-maven-app project is found.
