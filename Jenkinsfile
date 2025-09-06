pipeline {
    agent any

    environment {
        FRONTEND_DIR = 'ors10-frontend'
        TOMCAT_DIR = 'C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1\\webapps\\ORS'
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔍 Checking out frontend code..."
                deleteDir()
                git branch: 'main', url: 'https://github.com/rajatdhakad5/ors10-frontend.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo "📦 Installing npm packages..."
                    bat "npm install --legacy-peer-deps"
                }
            }
        }

        stage('Build') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo "🏗️ Building Angular project..."
                    bat "npx ng build --configuration production --base-href /ORS/ --aot=false --build-optimizer=false"
                }
            }
        }

        stage('Debug Paths') {
            steps {
                echo "📂 Checking important paths..."
                bat "echo Jenkins Workspace = %WORKSPACE%"
                bat "echo DIST_DIR = %WORKSPACE%\\${FRONTEND_DIR}\\dist"
                bat "echo TOMCAT_DIR = ${TOMCAT_DIR}"
            }
        }

        stage('List Dist Folder') {
            steps {
                dir("${FRONTEND_DIR}\\dist") {
                    echo "📂 Listing build output..."
                    bat "dir"
                }
            }
        }

        stage('Check Tomcat Folder Access') {
            steps {
                echo "📂 Checking Tomcat webapps folder access..."
                bat "dir \"${TOMCAT_DIR}\""
            }
        }

        stage('Clean Tomcat ORS Folder') {
            steps {
                echo "🧹 Cleaning existing deployed files from Tomcat ORS folder..."
                bat "rmdir /S /Q \"${TOMCAT_DIR}\" || echo Folder not found, skipping..."
                bat "mkdir \"${TOMCAT_DIR}\""
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying build to Tomcat..."
                bat """
                echo --- Before Copy ---
                dir "${TOMCAT_DIR}"

                echo --- Start Copy ---
                xcopy "${WORKSPACE}\\${FRONTEND_DIR}\\dist\\P10-UI\\*" "${TOMCAT_DIR}\\" /E /H /Y /I
                echo Exit Code = %ERRORLEVEL%

                echo --- After Copy ---
                dir "${TOMCAT_DIR}"
                """
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed. Check above for success/failure logs."
        }
    }
}
