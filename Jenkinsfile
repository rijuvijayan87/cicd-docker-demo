pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
    
    stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("rijuvijayan/train-schedule")
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
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
        stage('DeployToStaging') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    script {
                        //sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@$prod_ip \"sudo docker pull rijuvijayan/train-schedule:${env.BUILD_NUMBER}\""
                        sshagent(['my-ssh-key']) {
                            sh "$USERNAME@$prod_ip \"sudo docker pull rijuvijayan/train-schedule:${env.BUILD_NUMBER}\""
                        }
                    }
                }
            }
        }
    }
}
