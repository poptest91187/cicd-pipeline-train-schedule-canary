pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "nabilhermi/train-schedule"
        KUBECONFIG = credentials('kubeconfig-id')
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                    // Suppression des tags supplémentaires localement
                   // sh "docker rmi ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    //sh "docker rmi ${DOCKER_IMAGE_NAME}:latest"
                }
            }
        }



        stage('Prepare Kubernetes Deployment canary') {
           
            steps {
                script {
                    // Remplacement des variables dans le fichier YAML
                    sh """
                        sed -i 's|\\\$DOCKER_IMAGE_NAME|${DOCKER_IMAGE_NAME}|g' train-schedule-kube-canary.yml
                        sed -i 's|\\\$BUILD_NUMBER|${env.BUILD_NUMBER}|g' train-schedule-kube-canary.yml
                        sed -i 's|\\\$DOCKER_IMAGE_NAME|${DOCKER_IMAGE_NAME}|g' delete-train-schedule-kube-canary.yml
                        sed -i 's|\\\$BUILD_NUMBER|${env.BUILD_NUMBER}|g' delete-train-schedule-kube-canary.yml
                    """
                }
            }
        }

        stage('DeployToProduction canary') {
          
            steps {
               // input 'Deploy to Production?'
                //milestone(1)
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f train-schedule-kube-canary.yml'
                }
            }
        }
        
        stage('Prepare Kubernetes Deployment') {
            steps {
                script {
                    // Remplacement des variables dans le fichier YAML
                    sh """
                        sed -i 's|\\\$DOCKER_IMAGE_NAME|${DOCKER_IMAGE_NAME}|g' train-schedule-kube.yml
                        sed -i 's|\\\$BUILD_NUMBER|${env.BUILD_NUMBER}|g' train-schedule-kube.yml
                    """
                }
            }
        }


        stage('DeployToProduction') {
          
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f train-schedule-kube.yml'
                    sh 'kubectl apply -f delete-train-schedule-kube-canary.yml'
                }
            }
        }
        
    }
}
