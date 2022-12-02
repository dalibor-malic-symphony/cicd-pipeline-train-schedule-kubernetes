pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "dadomalic/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Install Docker') {
            when {
                equals(actual: sh(script: "docker -v", returnStdout: true), expected: 'docker: not found')
            }
            steps {
                echo '---- HELLO ----'
                /* 
                sh 'sudo apt-get update'
                sh 'sudo apt-get install ca-certificates curl gnupg lsb-release'
                sh 'sudo mkdir -p /etc/apt/keyrings'
                sh 'curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg'
                sh 'echo \"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null'
                sh 'sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin'
                */
            }
        }
        stage('Build Docker Image') {
            /* when {
                branch 'master'
            } */
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
            /* when {
                branch 'master'
            } */
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            /* when {
                branch 'master'
            } */
            steps {
                input 'Deploy to Production?'
                milestone(1)
                kubernetesDeploy(
                    kubeconfigId: 'kubeconfig',
                    configs: 'train-schedule-kube.yml',
                    enableConfigSubstitution: true
                )
            }
        }
    }
}
