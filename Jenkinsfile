pipeline {
    agent any

    stages {
        stage('Checkout Backend Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'jenkins-github-login',
                    url: 'https://github.com/nathanvo1489/ToDoList-BE.git'
            }
        }

        //// CI ////
        // must build with prefix username to push to Dockerhub
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t nathanvo1489/js_express_be_image:latest .'
            }
        }

        stage('Dockerhub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jenkins-dockerhub-login',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                    '''
                }
            }
        }

        stage('Push Img to Dockerhub') {
            steps {
                sh 'docker push nathanvo1489/js_express_be_image:latest'
            }
        }

        //// CD in server run ////
        // Rmb to check docker-compose.yml & .env file at the newest version //
        stage('Deploy Container') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ssh-login',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    ),
                    string(
                        credentialsId: 'ip-server-run-be',
                        variable: 'IP_SERVER_RUN_BE')
                ]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i ${SSH_KEY} ${SSH_USER}@${IP_SERVER_RUN_BE} "
                            ls -la
                            cd /home/ubuntu/ToDoList-BE/docker-compose/
                            docker compose pull js_express_be
                            docker compose up -d --force-recreate js_express_be
                            docker ps
                            docker image prune -f
                        "
                    '''
                }
            }
        }
    }
}

