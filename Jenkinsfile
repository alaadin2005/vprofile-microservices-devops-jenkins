pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:

  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - /kaniko/executor
    args:
    - "--dockerfile=Docker-files/app/Dockerfile"
    - "--context=git://github.com/alaadin2005/vprofile-microservices-devops-jenkins.git"
    - "--destination=alaadin2005/vprofileapp:1"
    - "--skip-tls-verify"

    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true

  - name: jnlp
    image: jenkins/inbound-agent:latest
"""
        }
    }

    environment {
        DOCKERHUB_CREDS = credentials('dockerhub')
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/alaadin2005/vprofile-microservices-devops-jenkins.git',
                    branch: 'main'
            }
        }

        stage('Build & Push Image') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                    --dockerfile=Docker-files/app/Dockerfile \
                    --context=$WORKSPACE \
                    --destination=alaadin2005/vprofileapp:1 \
                    --skip-tls-verify
                    '''
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                container('kubectl') {
                    sh '''
                    kubectl apply -f k8s/
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Success"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
