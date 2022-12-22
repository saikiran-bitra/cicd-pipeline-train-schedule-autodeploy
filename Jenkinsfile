pipeline {
    agent any
    environment {
        //be sure to replace "saikiran989" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "saikiran989/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh '/opt/gradle/gradle-5.0/bin/gradle build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        
        stage('ID') {
           
            steps {
                sh 'echo $USER'
            }
        }
        stage('Build Docker Image') {
            when {
                branch "master"
            }
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
            when {
                branch "master"
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch "master"
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                sh 'export CANARY_REPLICAS='+CANARY_REPLICAS+' ; envstub < train-schedule-kube-canary.yml | kubectl apply -f -'
            }
        }
        stage('DeployToProduction') {
            when {
                branch "master"
            }
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                sh 'export CANARY_REPLICAS='+CANARY_REPLICAS+' ; envstub < train-schedule-kube-canary.yml | kubectl apply -f -'
                sh 'export CANARY_REPLICAS='+CANARY_REPLICAS+' ; envstub < train-schedule-kube.yml | kubectl apply -f -'
            }
        }
    }
}
