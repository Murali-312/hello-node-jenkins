pipeline {
    agent any

    environment {
        REMOTE_USER = "murali"
        REMOTE_HOST = "192.168.1.118"
        APP_NAME    = "hello-node-jenkins"
        DEPLOY_DIR  = "/home/murali/deployments/${APP_NAME}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code"
                git branch: 'main',
                    url: 'https://github.com/Murali-312/hello-node-jenkins.git'
            }
        }

        stage('Package Application') {
            steps {
                echo "📦 Packaging application"
                sh '''
                  rm -f ${APP_NAME}.zip
                  zip -r ${APP_NAME}.zip . -x .git/* -x node_modules/*
                '''
            }
        }

        stage('Copy Artifact to Remote Server') {
            steps {
                echo "🚚 Copying artifact to remote server"
                sh '''
                  scp ${APP_NAME}.zip ${REMOTE_USER}@${REMOTE_HOST}:~/${APP_NAME}.zip
                '''
            }
        }

        stage('Deploy & Run on Remote Server') {
            steps {
                echo "🚀 Deploying application on remote server"
                sh """
ssh ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
set -e

echo "📂 Preparing deployment directory"
mkdir -p ${DEPLOY_DIR}

echo "📦 Unzipping application"
unzip -o ~/${APP_NAME}.zip -d ${DEPLOY_DIR}

cd ${DEPLOY_DIR}

echo "🧪 Node version:"
node -v
echo "🧪 NPM version:"
npm -v

echo "📥 Installing dependencies"
npm install --omit=dev

echo "🔁 Restarting application via PM2"
pm2 delete ${APP_NAME} || true
pm2 start app.js --name ${APP_NAME}
pm2 save

echo "✅ Application deployed successfully"
EOF
                """
            }
        }
    }

    /* 🔔 LOCAL NOTIFICATIONS (Option 1 + Option 2) */
    post {
        success {
            echo "✅ BUILD SUCCESS: ${APP_NAME} deployed on ${REMOTE_HOST}"
        }
        failure {
            echo "❌ BUILD FAILED: Check console logs for details"
        }
        always {
            echo "ℹ️ Pipeline finished at ${new Date()}"
        }
    }
}

