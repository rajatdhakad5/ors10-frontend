pipeline {
    agent any

    environment {
        // Paths
        WORKSPACE_DIR = "${env.WORKSPACE}"
        DIST_DIR = "${WORKSPACE_DIR}\\ors10-frontend\\dist\\ors10-frontend"
        TOMCAT_DIR = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1\\webapps\\ORS"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "🔑 Checking out frontend code..."
                deleteDir()
                git branch: 'main',
                    url: 'https://github.com/rajatdhakad5/ors10-frontend.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("ors10-frontend") {
                    echo "📦 Installing npm packages..."
                    bat "npm install --legacy-peer-deps"
                }
            }
        }

        stage('Build') {
            steps {
                dir("ors10-frontend") {
                    echo "🏗️ Building Angular project..."
                    bat "npx ng build --configuration production --base-href /ORS/ --aot=false --build-optimizer=false"
                }
            }
        }

        stage('Debug Paths') {
            steps {
                echo "🖊 Checking important paths..."
                bat "echo Jenkins Workspace = %WORKSPACE%"
                bat "echo DIST_DIR = %DIST_DIR%"
                bat "echo TOMCAT_DIR = %TOMCAT_DIR%"
            }
        }

        stage('List Dist Folder') {
            steps {
                dir("ors10-frontend/dist/ors10-frontend") {
                    echo "📂 Listing build output..."
                    bat "dir"
                }
            }
        }

        stage('Check Tomcat Folder Access') {
            steps {
                echo "📂 Checking Tomcat webapps folder access..."
                bat "dir \"%TOMCAT_DIR%\""
            }
        }

        stage('Clean Tomcat ORS Folder') {
            steps {
                echo "🧹 Cleaning existing deployed files from Tomcat ORS folder..."
                bat "rmdir /S /Q \"%TOMCAT_DIR%\" || echo Folder not found, skipping..."
                bat "mkdir \"%TOMCAT_DIR%\""
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying build to Tomcat..."
                bat "echo --- Before Copy ---"
                bat "dir \"%TOMCAT_DIR%\""

                bat "echo --- Start Copy ---"
                bat "xcopy \"%DIST_DIR%\\*\" \"%TOMCAT_DIR%\\\" /E /H /Y /I"

                bat "echo --- After Copy ---"
                bat "dir \"%TOMCAT_DIR%\""
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed. Check above for success/failure logs."
        }
    }
}
