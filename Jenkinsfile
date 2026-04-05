// pipeline {
//     agent any

//     stages {

//         stage('Checkout from GitHub') {
//             steps {
//                 git branch: 'master',
//                     url: 'https://github.com/sathvik-dande/node-k8s-app.git'
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 sh 'npm install'
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 sh '''
//                 docker build -t my-k8s-app:${BUILD_NUMBER} .
//                 docker tag my-k8s-app:${BUILD_NUMBER} sathvik-dande/my-k8s-app:latest
//                 '''
//             }
//         }

//         stage('Push Docker Image') {
//             steps {
//                 sh 'docker push sathvik-dande/my-k8s-app:latest'
//             }
//         }

//         stage('Start Minikube if not running') {
//             steps {
//                 sh '''
//                 if ! minikube status | grep -q "apiserver: Running"; then
//                     echo "Minikube is not running. Starting now..."
//                     minikube start --driver=docker --memory=2048 --cpus=2
//                 fi
//                 '''
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 sh '''
//                 # Load latest image into Minikube
//                 # minikube image load sathvik-dande/my-k8s-app:latest

//                 # Apply manifests
//                 minikube kubectl -- apply -f k8s/deployment.yaml
//                 minikube kubectl -- apply -f k8s/service.yaml
//                 minikube service my-k8s-app-service
//                 '''
//             }
//         }
//     }
// }
pipeline {
    agent any

    stages {

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t my-k8s-app:${BUILD_NUMBER} .
                docker tag my-k8s-app:${BUILD_NUMBER} sathvik-dande/my-k8s-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push sathvik-dande/my-k8s-app:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                # Stop old container if exists
                docker stop my-app || true
                docker rm my-app || true

                # Run new container
                docker run -d -p 3001:3000 --name my-app sathvik-dande/my-k8s-app:9
                '''
            }
        }
    }
}
