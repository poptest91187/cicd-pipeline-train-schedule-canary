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

      
        stage('DeployToProduction') {
          
            steps {
               // input 'Deploy to Production?'
                //milestone(1)
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f train-schedule-kube.yml'
                }
            }
        }
        
    }
}
