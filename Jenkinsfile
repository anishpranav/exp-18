pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "yourdockerhubusername/flaskapp"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/anishpranav/exp-18.git'
            }
        }


        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    sh """
                        echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    writeFile file: 'deployment.yaml', text: """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flaskapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flaskapp
  template:
    metadata:
      labels:
        app: flaskapp
    spec:
      containers:
      - name: flaskapp
        image: ${DOCKER_IMAGE}:${BUILD_NUMBER}
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: flaskapp-service
spec:
  type: NodePort
  selector:
    app: flaskapp
  ports:
  - port: 5000
    targetPort: 5000
    nodePort: 30001
                    """
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
