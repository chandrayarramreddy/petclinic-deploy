pipeline {
  agent any

  environment {
    REGISTRY = "docker.io/chandray"
    //IMAGE = "${REGISTRY}/petclinic"
    IMAGE = "${REGISTRY}/spring-petclinic:4.0.0-SNAPSHOT"
    VERSION = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { 
        script {
        def branchToCheckout = 'main' // Define the branch name
        checkout([$class: 'GitSCM', 
          branches: [[name: "refs/heads/${branchToCheckout}"]], // Specify the branch
          userRemoteConfigs: [[credentialsId: 'git_vmtoken', url: 'https://github.com/spring-projects/spring-petclinic.git']], // Your repository details
          extensions: []]) }
      }
    }

    stage('Build & Unit Test') {
      steps { //sh './mvnw clean package -DskipTests=false'
      sh './mvnw clean package -DskipTests' }
    }

    stage('Build Docker Image') {
      steps { //sh "docker build -t $IMAGE:$VERSION ." 
      sh './mvnw spring-boot:build-image' }
    }

    stage('Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
          sh "echo $PASS | docker login -u $USER --password-stdin"
          //sh "docker push $IMAGE:$VERSION"
           sh "docker tag $IMAGE $IMAGE"
           sh "docker push $IMAGE"
        }
      }
    }

    stage('Update Manifest Repo') {
      steps {
        sh '''
          git clone https://github.com/chandrayarramreddy/petclinic-deploy.git
          cd petclinic-deploy
          sed -i "s|image:.*|image: $IMAGE:$VERSION|" deployment.yaml
          git config user.name "chandra"
          git config user.email "chandra.yarramreddy@gmail.com"
          git commit -am "Deploy build $VERSION"
          git push origin main
        '''
      }
    }
  }
}
