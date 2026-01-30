pipeline {
    agent any

    environment {
        REMOTE_USER = "murali"
        REMOTE_HOST = "192.168.1.118"
        APP_NAME = "hello-node-jenkins"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Murali-312/hello-node-jenkins.git'
            }
        }

        stage('Zip Source Code') {
            steps {
                sh 'zip -r ${APP_NAME}.zip .'
            }
        }

        stage('Copy to Remote Server') {
            steps {
                sh 'scp ${APP_NAME}.zip ${REMOTE_USER}@${REMOTE_HOST}:~/${APP_NAME}.zip'
            }
        }

        stage('Deploy & Run on Remote Server') {
            steps {
                sh '''
                ssh ${REMOTE_USER}@${REMOTE_HOST} << EOF
                mkdir -p ~/deployments/${APP_NAME}
                unzip -o ~/${APP_NAME}.zip -d ~/deployments/${APP_NAME}
                cd ~/deployments/${APP_NAME}
                npm install
                pm2 delete ${APP_NAME} || true
                pm2 start app.js --name ${APP_NAME}
                pm2 save
                EOF
                '''
            }
        }
    }
}

