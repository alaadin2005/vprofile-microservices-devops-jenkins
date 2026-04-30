pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins

  containers:

  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    args: ["--help"]
    command:
      - cat
    tty: true
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker

  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - cat
    tty: true

  volumes:
    - name: docker-config
      secret:
        secretName: dockerhub
'''
        }
    }

    environment {
        DOCKERHUB_USER = 'alaadin2005'
        APP_IMAGE = 'vprofileapp'
        DB_IMAGE = 'vprofiledb'
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/abdelrahmanonline4/dockerized-microservices.git'
            }
        }

        stage('Build App Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --dockerfile=Docker-files/app/Dockerfile \
                    --context=\$(pwd) \
                    --destination=${DOCKERHUB_USER}/${APP_IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Build DB Image') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                    --dockerfile=Docker-files/db/Dockerfile \
                    --context=\$(pwd) \
                    --destination=${DOCKERHUB_USER}/${DB_IMAGE}:${TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
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
