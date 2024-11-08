pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
        PWD = credentials('pwd-ubuser')
    }
    stages {
        stage('Setup') {
            steps {
                sh "pip install -r requirements.txt"
            }
        }
        stage('Test') {
            steps {
                sh "pytest"
            }
        }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip  ${username}@${SERVER_IP}:/home/ubuser/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/ubuser/myapp.zip -d /home/ubuser/app/
                        source app/venv/bin/activate
                        cd /home/ubuser/app/
                        pip install -r requirements.txt
                        echo "----- fin pip " 
                        sudo systemctl restart flaskapp.service
EOF
                    '''
                }
            }
        }
       
        
       
        
    }
}