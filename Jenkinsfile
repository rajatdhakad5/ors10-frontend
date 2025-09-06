pipeline {
    agent any

    environment {
        NODE_HOME = "C:\\Program Files\\nodejs"
        JAVA_HOME = "C:\\Program Files\\Java\\jdk-11.0.15.1"
        PATH = "${env.NODE_HOME};${env.JAVA_HOME}\\bin;${env.PATH}"
        FRONTEND_DIR = "ors10-frontend"
        DIST_DIR = "dist\\ors10-frontend"
        TOMCAT_DIR = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1\\webapps\\ORS"
        TOMCAT_BIN = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1\\bin"
        CATALINA_HOME = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out frontend code..."
                deleteDir()
                git branch: 'main', url: 'https://github.com/rajatdhakad5/ors10-frontend.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    echo "📦 Installing npm packages..."
                    bat "\"${env.NODE_HOME}\\npm.cmd\" install --legacy-peer-deps"
                }
            }
        }

        stage('Build') {
            steps {
                dir("${env.FRONTEND_DIR}") {
                    echo "🏗 Building Angular project..."
                    bat "\"${env.NODE_HOME}\\npx.cmd\" ng build --configuration production --base-href /ORS/ --aot=false --build-optimizer=false"
                }
            }
        }

        stage('Clean Tomcat ORS Folder') {
            steps {
                echo "🧹 Cleaning existing deployed files from Tomcat ORS folder..."
                bat "rmdir /S /Q \"${env.TOMCAT_DIR}\" || echo Folder not found, skipping..."
                bat "mkdir \"${env.TOMCAT_DIR}\""
            }
        }

        stage('Deploy') {
            steps {
                echo "🚀 Deploying build to Tomcat..."
                bat "xcopy \"${env.DIST_DIR}\" \"${env.TOMCAT_DIR}\" /E /H /Y /I"
            }
        }

        stage('Restart Tomcat') {
            steps {
                echo "♻ Restarting Tomcat server..."
                script {
                    bat "%CATALINA_HOME%\\bin\\shutdown.bat || echo Tomcat not running"
                    def startStatus = bat(
                        script: """
                            set CATALINA_HOME=${env.CATALINA_HOME}
                            set JAVA_HOME=${env.JAVA_HOME}
                            %CATALINA_HOME%\\bin\\startup.bat
                        """,
                        returnStatus: true
                    )

                    if (startStatus != 0) {
                        error("❌ Tomcat failed to start. Please check manually.")
                    } else {
                        echo "✅ Tomcat started successfully."
                    }
                }
            }
        }
    }

    post {
        always {
            echo "✅ Pipeline completed. Check above for success/failure logs."
        }
    }
}
