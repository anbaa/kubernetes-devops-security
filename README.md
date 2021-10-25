# kubernetes-devops-security

## Fork and Clone this Repo

## Clone to Desktop and VM

## NodeJS Microservice - Docker Image -
`docker run -p 8787:5000 siddharth67/node-service:v1`

`curl localhost:8787/plusone/99`
 
## NodeJS Microservice - Kubernetes Deployment -
`kubectl create deploy node-app --image siddharth67/node-service:v1`

`kubectl expose deploy node-app --name node-service --port 5000 --type ClusterIP`

`curl node-service-ip:5000/plusone/99`

#docker

You may experience following error while pushing image "WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.39/auth: dial unix /var/run/docker.sock: connect: permission denied"


Please run following commands on the instance

usermod -aG docker jenkins
chmod 777 /var/run/docker.sock
 
and update following jenkins files as per instructions from "https://github.com/darinpope/dockerhub-example/blob/01-no-plugins-necessary/Jenkinsfile" https://www.youtube.com/watch?v=alQQ84M4CYU

'''
pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub')
  }

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and Jacoco') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Build') {
      steps {
        sh 'docker build -t anbazhagandkr/numeric-app:""$GIT_COMMIT"" .'
      }
    }
    stage('Login') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push anbazhagandkr/numeric-app:""$GIT_COMMIT""'
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
  
}
'''
#fix GIT
