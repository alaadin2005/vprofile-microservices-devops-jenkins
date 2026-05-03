pipeline {
  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest

  - name: docker
    image: docker:27-cli
    command: ["cat"]
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock

  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["cat"]
    tty: true

  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
    }
  }

  environment {
    IMAGE = "alaadin2005/vprofileapp:1"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main',
        url: 'https://github.com/alaadin2005/vprofile-microservices-devops-jenkins.git'
      }
    }

    stage('Docker Build') {
      steps {
        container('docker') {
          sh "docker build -t $IMAGE Docker-files/app"
        }
      }
    }

    stage('Docker Login') {
      steps {
        container('docker') {
          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh '''
              echo $PASS | docker login -u $USER --password-stdin
            '''
          }
        }
      }
    }

    stage('Push Image') {
      steps {
        container('docker') {
          sh "docker push $IMAGE"
        }
      }
    }

    stage('Deploy to K8s') {
      steps {
        container('kubectl') {
          sh """
            kubectl apply -f k8s/
          """
        }
      }
    }
  }

  post {
    always {
      echo "Pipeline Finished"
    }
  }
}
