pipeline {
    agent any
    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DatlaBharath/HelloService-Uni'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "sakthisiddu1/helloservice-uni:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh 'echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin'
                        def imageName = "sakthisiddu1/helloservice-uni:${env.BUILD_NUMBER}"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloservice-uni-deployment
  labels:
    app: helloservice-uni
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloservice-uni
  template:
    metadata:
      labels:
        app: helloservice-uni
    spec:
      containers:
      - name: helloservice-uni
        image: sakthisiddu1/helloservice-uni:${env.BUILD_NUMBER}
        ports:
        - containerPort: 5000
"""

                    def serviceYaml = """
apiVersion: v1
kind: Service
metadata:
  name: helloservice-uni-service
spec:
  selector:
    app: helloservice-uni
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
    nodePort: 30007
  type: NodePort
"""

                    sh """echo "$deploymentYaml" > deployment.yaml"""
                    sh """echo "$serviceYaml" > service.yaml"""

                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@43.205.114.246 "kubectl apply -f -" < deployment.yaml'
                    sh 'ssh -i /var/test.pem -o StrictHostKeyChecking=no ubuntu@43.205.114.246 "kubectl apply -f -" < service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment was successful'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}