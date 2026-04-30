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
    image: gcr.io/kaniko-project/executor:v1.23.2
    command: ["/busybox/sh", "-c"]
    args: ["cat"]
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker

  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["/bin/sh", "-c"]
    args: ["cat"]

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
                    sh '''
                    /kaniko/executor \
                    --dockerfile=Docker-files/app/Dockerfile \
                    --context=$PWD \
                    --destination=alaadin2005/vprofileapp:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Build DB Image') {
            steps {
                container('kaniko') {
                    sh '''
                    /kaniko/executor \
                    --dockerfile=Docker-files/db/Dockerfile \
                    --context=$PWD \
                    --destination=alaadin2005/vprofiledb:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Deploy') {
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
            echo "✅ Success"
        }
        failure {
            echo "❌ Failed"
        }
    }
}
