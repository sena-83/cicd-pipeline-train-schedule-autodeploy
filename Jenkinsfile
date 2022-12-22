pipeline {
    agent any
    tools {
        jdk 'java8'
    }
    environment {
        DOCKER_IMAGE_NAME = "sena83/train-schedule"
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
            //when {
             //   branch 'master'
           // }
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Build docker image successful!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            //when {
              //  branch 'master'
            //}
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHub') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            //when {
              //  branch 'master'
            //}
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                //withKubeConfig([credentialsId: 'kubernetes-admin', serverUrl: 'https://172.31.5.21:6443']) {
                 //sh 'kubectl apply -f train-schedule-kube-canary.yml'
                //}
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml'
                )
            }
        }
        stage('DeployToProduction') {
            //when {
              //  branch 'master'
            //}
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube-canary.yml',
                    enableConfigSubstitution: true
                )
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
