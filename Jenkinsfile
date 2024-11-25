pipeline {
    agent any

    environment {
        KOPS_STATE_STORE = 's3://k8projectnov17' // Set your state store path
        CLUSTER_NAME = 'vineet.k8s.local'         // Set the cluster name
    }

    stages {
        stage('Prepare Workspace') {
            steps {
                sh 'mkdir -p /var/lib/jenkins/workspace/vineetproj'
                sh 'echo "FROM ubuntu:latest" > /var/lib/jenkins/workspace/vineetproj/Dockerfile'
            }
        }

        stage('Pull Code From GitHub') {
            steps {
                git url: 'https://github.com/vineet-020396/K8_Project_Nov_17.git', credentialsId: 'github-pat-vineet1', branch: 'main'
            }
        }

        stage('Build the Docker Image') {
            steps {
                script {
                    def imageName = "vineet020396/vineetimage"
                    def buildTag = "${env.BUILD_NUMBER}"

                    sh 'sudo docker build -t vineet /var/lib/jenkins/workspace/vineetproj'
                    sh "sudo docker tag vineet ${imageName}:latest"
                    sh "sudo docker tag vineet ${imageName}:${buildTag}"
                }
            }
        }

        stage('Push the Docker Image') {
            steps {
                script {
                    def imageName = "vineet020396/vineetimage"
                    def buildTag = "${env.BUILD_NUMBER}"

                    sh "sudo docker image push ${imageName}:latest"
                    sh "sudo docker image push ${imageName}:${buildTag}"
                }
            }
        }

        stage('Create Kubernetes Cluster with Kops') {
            steps {
                script {
                    // Create the cluster using Kops
                    sh """
                        kops create cluster --name ${env.CLUSTER_NAME} --state ${env.KOPS_STATE_STORE} --zones us-east-1a --node-count 3 --node-size t3.medium --master-size t3.medium --dns private --yes
                    """
                }
            }
        }

        stage('Export Kubeconfig') {
            steps {
                script {
                    // Export the kubeconfig to interact with the cluster
                    sh "kops export kubeconfig --name ${env.CLUSTER_NAME} --state ${env.KOPS_STATE_STORE}"
                }
            }
        }

        stage('Create Kubernetes Pod YAML Dynamically') {
            steps {
                script {
                    def podYaml = """
                    apiVersion: v1
                    kind: Pod
                    metadata:
                      name: loadbalancer-pod
                    spec:
                      containers:
                        - name: loadbalancer
                          image: vineet020396/vineetimage:${env.BUILD_NUMBER}
                          ports:
                            - containerPort: 80
                    """
                    writeFile(file: '/var/lib/jenkins/workspace/vineetproj/pod.yaml', text: podYaml)
                }
            }
        }

        stage('Deploy on Kubernetes') {
            steps {
                // Apply the dynamically generated Kubernetes manifest
                sh 'sudo kubectl apply -f /var/lib/jenkins/workspace/vineetproj/pod.yaml'
                sh 'sudo kubectl rollout restart deployment loadbalancer-pod'
            }
        }
    }
}
