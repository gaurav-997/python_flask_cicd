pipeline{
    agent any
    environment{
        SERVER_IP = credential("server_ip")
    }

    stages{
        stage("install dependicies"){
            steps{
                script{
                    sh " pip install -r requirmnets.txt"
                }
            }
        }
        stage("test"){
            steps{
                script{
                    sh "pytest"
                }
            }
        }
        stage("package code"){
            steps{
                script{
                    sh "zip -r myapp.zip ./* -x '*.git*'"
                    sh " ls -lrt"
                }
            }
        }
        stage("copy & deploy to server"){
            steps{
                script{
                    withCredentials(sshUserPrivateKey[credentialID:"ssh-key" , keyfileVariable:'ssh-key',
                    usernameVariable:'username'])
                    sh ''' 
                    scp -r  -i ${ssh-key} myapp.zip ${username}@${SERVER_IP}:/home/ubuntu
                    ssh -i ${ssh-key} ${username}@${SERVER_IP} << 
                    EOF
                        unzip -o  /home/ubuntu/myappp.zip -d /home/ubuntu/app
                        source app/venv/bin/activate
                        cd /home/ubuntu/app
                        pip install -r requirments.txt
                        sudo systemctl start flaskapp.service
                    EOF

                    '''

                }
            }
        }
    }
}