pipeline {
    agent any

    environment {
        REMOTE_USER = "murali"
        REMOTE_HOST = "192.168.1.118"
        APP_NAME    = "hello-node-jenkins"
        DEPLOY_DIR  = "/home/murali/deployments/${APP_NAME}"
        NVM_DIR     = "/home/murali/.nvm"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Murali-312/hello-node-jenkins.git'
            }
        }

        stage('Install Dependencies (CI)') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        def scannerHome = tool 'SonarScanner'
                        sh """
                          ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=hello-node-jenkins \
                            -Dsonar.projectName=hello-node-jenkins \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=node_modules/**,.git/**
                        """
                    }
                }
            }
        }

        /* =========================
           SNYK SECURITY SCAN
           ========================= */
        stage('Snyk Security Scan') {
            environment {
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                sh 'snyk test --severity-threshold=high'
            }
        }

        /* =========================
           SNYK MONITOR (REPORTING)
           ========================= */
        stage('Snyk Monitor') {
            environment {
                SNYK_TOKEN = credentials('snyk-token')
            }
            steps {
                sh 'snyk monitor'
            }
        }

        stage('Zip Source Code') {
            steps {
                sh '''
                  rm -f ${APP_NAME}.zip
                  zip -r ${APP_NAME}.zip . -x .git/* -x node_modules/*
                '''
            }
        }

        stage('Copy to Remote Server') {
            steps {
                sh '''
                  scp ${APP_NAME}.zip ${REMOTE_USER}@${REMOTE_HOST}:~/${APP_NAME}.zip
                '''
            }
        }

        stage('Deploy & Run on Remote Server') {
            steps {
                sh """
ssh ${REMOTE_USER}@${REMOTE_HOST} << 'EOF'
set -e

export NVM_DIR="${NVM_DIR}"
[ -s "\$NVM_DIR/nvm.sh" ] && . "\$NVM_DIR/nvm.sh"

mkdir -p ${DEPLOY_DIR}
unzip -o ~/${APP_NAME}.zip -d ${DEPLOY_DIR}
cd ${DEPLOY_DIR}

node -v
npm -v
npm install --omit=dev

pm2 delete ${APP_NAME} || true
pm2 start app.js --name ${APP_NAME}
pm2 save
EOF
                """
            }
        }
    }
}

