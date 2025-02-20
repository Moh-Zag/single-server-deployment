pipeline {
    agent any

    environment {
        SERVER_IP = credentials('prod-server-ip')
    }
    stages {
        // stage('Setup') {
        //     steps {
        //         sh "pip install -r requirements.txt"
        //     }
        // }
        // stage('Test') {
        //     steps {
        //         sh "pytest"
        //     }
        // }

        stage('Package code') {
            steps {
                sh "zip -r myapp.zip ./* -x '*.git*'"
                sh "ls -lart"
            }
        }

        stage('Deploy to Prod') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'prod-server-ssh-key', keyFileVariable: 'MY_SSH_KEY', usernameVariable: 'username')]) {
                    sh '''
                    scp -i $MY_SSH_KEY -o StrictHostKeyChecking=no myapp.zip  ${username}@${SERVER_IP}:/home/sysuser/
                    ssh -i $MY_SSH_KEY -o StrictHostKeyChecking=no ${username}@${SERVER_IP} << EOF
                        unzip -o /home/sysuser/myapp.zip -d /home/sysuser/app/
                        source app/venv/bin/activate
                        cd /home/sysuser/app/
                        pwd
                        ls -al
                        pip install -r requirements.txt
EOF
                    '''
                }
            withCredentials([usernamePassword(credentialsId: 'prod-server-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                sh 'echo ${PASSWORD} | sudo systemctl restart flaskapp.service --stdin' }
            }
        }

    }
}