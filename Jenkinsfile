pipeline {
    agent any
    environment {
        SERVER_IP = credentials("server-ip")
    }

    stages {
        stage("git checkout"){
            steps{
                script{
                    git branch: 'main', 
                url: 'https://github.com/gaurav-997/python_flask_cicd.git'
                }
            }
        }
        stage("Install Dependencies") {
            steps {
                script {
                    sh '''
                    # Ensure Jenkins user can create venv
                    sudo python3 -m venv /home/ubuntu/app/venv

                    # Set correct permissions
                    sudo chown -R $(whoami) /home/ubuntu/app/venv

                    # Activate virtual environment and install dependencies
                    source /home/ubuntu/app/venv/bin/activate
                    pip3 install -r /home/ubuntu/app/requirements.txt '''
                }
            }
        }
        stage("Run Tests") {
            steps {
                script {
                    sh "pytest"
                }
            }
        }
        stage("Package Code") {
            steps {
                script {
                    sh "zip -r myapp.zip ./* -x '*.git*'"
                    sh "ls -lrt"
                }
            }
        }
        stage("Deploy to Server") {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'SSH_KEY', usernameVariable: 'USERNAME')]) {
                        sh '''
                        echo "Transferring files to ${SERVER_IP}..."
                        scp -i ${SSH_KEY} myapp.zip ${USERNAME}@${SERVER_IP}:/home/ubuntu

                        echo "Deploying on ${SERVER_IP}..."
                        ssh -i ${SSH_KEY} ${USERNAME}@${SERVER_IP} << 'EOF'
                            echo "Unzipping application..."
                            unzip -o /home/ubuntu/myapp.zip -d /home/ubuntu/app

                            echo "Activating virtual environment..."
                            source /home/ubuntu/app/venv/bin/activate

                            echo "Installing dependencies..."
                            pip3 install -r /home/ubuntu/app/requirements.txt

                            echo "Restarting Flask application..."
                            sudo systemctl restart flaskapp.service
                        EOF
                        '''
                    }
                }
            }
        }
    }
}
