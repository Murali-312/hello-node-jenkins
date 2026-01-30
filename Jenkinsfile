pipeline {
    agent any

    environment {
        REMOTE_USER = "murali"
        REMOTE_HOST = "192.168.1.118"
        APP_NAME    = "hello-node-jenkins"
        DEPLOY_DIR  = "/home/murali/deployments"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Murali-312/hello-node-jenkins.git'
            }
        }

        stage('Package Application') {
            steps {
                sh '''
                rm -f ${APP_NAME}.zip
                zip -r ${APP_NAME}.zip . \
                    -x ".git/*" \
                    -x "node_modules/*"
                '''
            }
        }

        stage('Copy Artifact to Remote Server') {
            steps {
                sh '''
                scp ${APP_NAME}.zip \
                ${REMOTE_USER}@${REMOTE_HOST}:~/${APP_NAME}.zip
                '''
            }
        }

        stage('Deploy & Run on Remote Server') {
            steps {
                sh '''
                ssh ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
                set -e

                echo "📦 Deploying ${APP_NAME}"

                mkdir -p ${DEPLOY_DIR}/${APP_NAME}
                unzip -o ~/${APP_NAME}.zip -d ${DEPLOY_DIR}/${APP_NAME}
                cd ${DEPLOY_DIR}/${APP_NAME}

                # ---- Load NVM explicitly ----
                export NVM_DIR="$HOME/.nvm"
                [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

                echo "🧪 Node version:"
                node -v
                echo "🧪 NPM version:"
                npm -v

                npm install --production

                pm2 delete ${APP_NAME} || true
                pm2 start app.js --name ${APP_NAME}
                pm2 save

                echo "✅ Deployment completed"
                EOF
                '''
            }
        }
    }
}

