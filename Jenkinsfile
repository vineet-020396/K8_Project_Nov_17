pipeline {
    agent any

    stages {
        stage('Pull Code From GitHub') {
            steps {
                git 'https://github.com/Alvinbsi/K8s_Project_NOV12.git'
            }
        }
        stage('Build the Docker image') {
            steps {
                sh 'sudo docker build -t alvink8simage /var/lib/jenkins/workspace/k8s_build_nov13'
                sh 'sudo docker tag alvink8simage alvinselva/alvink8simage:latest'
                sh 'sudo docker tag alvink8simage alvinselva/alvink8simage:${BUILD_NUMBER}'
            }
        }
        stage('Push the Docker image') {
            steps {
                sh 'sudo docker image push alvinselva/alvink8simage:latest'
                sh 'sudo docker image push alvinselva/alvink8simage:${BUILD_NUMBER}'
            }
        }
        stage('Deploy on Kubernetes') {
            steps {
                sh 'sudo kubectl apply -f /var/lib/jenkins/workspace/k8s_build_nov13/pod.yaml'
                sh 'sudo kubectl rollout restart deployment loadbalancer-pod'
            }
        }
    }
}
