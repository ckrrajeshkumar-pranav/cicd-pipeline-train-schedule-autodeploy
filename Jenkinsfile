pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "ckrrajeshkumar/train-schedule"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ckrrajeshkumar-pranav/cicd-pipeline-train-schedule-autodeploy.git'
            }
        }
        stage('Build Application') {
            steps {
                script {
                    sh './gradlew build --no-daemon'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.DOCKER_TAG = "${commitHash}"
                }
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                     sh """
                    kubernetesDeploy(configs: "deployment.yaml", "service.yaml", kubeconfigID: "kube")
                    kubectl config use-context your-cluster-context
                    kubectl set image deployment/train-schedule train-schedule=$DOCKER_IMAGE:$DOCKER_TAG
                    kubectl rollout status deployment/train-schedule
                    """
                }
            }
        }
    }
}

