pipeline {
    agent any

    tools {
        nodejs 'Node16' // Make sure this matches your Jenkins NodeJS tool name
    }

    environment {
        PORT = '3010'
        HOST = '0.0.0.0'
        APP_NAME = 'FIRMSFRONTEND'
        APP_DIR = '/var/lib/jenkins/.jenkins/workspace/FIRMSFRONTEND'
        PM2_HOME = '/var/lib/jenkins/.pm2'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
    steps {
        sh 'npm ci --legacy-peer-deps'
    }
}




        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Clean Workspace') {
    steps {
        sh 'rm -rf .next'
    }
}

        stage('Build App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy with PM2') {
            steps {
                sh '''
                  export PM2_HOME=${PM2_HOME}

                  if pm2 describe ${APP_NAME} > /dev/null; then
                    echo "App exists. Restarting..."
                    pm2 restart ${APP_NAME}
                  else
                    echo "App not found. Starting fresh instance..."
                    pm2 start node_modules/next/dist/bin/next \
                      --name ${APP_NAME} \
                      -- start -p ${PORT} -H ${HOST} \
                      --cwd ${APP_DIR} \
                      -i 1
                  fi

                  pm2 save
                  pm2 status
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ Build failed. Attempting to revert to last successful commit..."

            sh '''
              if [ -n "$GIT_PREVIOUS_SUCCESSFUL_COMMIT" ]; then
                echo "Reverting to commit: $GIT_PREVIOUS_SUCCESSFUL_COMMIT"
                git fetch --all
                git checkout $GIT_PREVIOUS_SUCCESSFUL_COMMIT

                npm install
                npm run build

                export PM2_HOME=${PM2_HOME}

                if pm2 describe ${APP_NAME} > /dev/null; then
                  pm2 restart ${APP_NAME}
                else
                  pm2 start node_modules/next/dist/bin/next \
                    --name ${APP_NAME} \
                    -- start -p ${PORT} -H ${HOST} \
                    --cwd ${APP_DIR} \
                    -i 1
                fi

                pm2 save
              else
                echo "⚠ No previous successful build found. Cannot revert."
              fi
            '''
            echo "🚨 Revert process completed."
        }
    }
}

