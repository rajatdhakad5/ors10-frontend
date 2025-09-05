pipeline {
    agent any

    environment {
        NODE_HOME = "C:\\Program Files\\nodejs"
        JAVA_HOME = "C:\\Program Files\\Java\\jdk-11.0.15.1"
        PATH = "${env.NODE_HOME};${env.JAVA_HOME}\\bin;${env.PATH}"

        FRONTEND_REPO = 'https://github.com/rajatdhakad5/ors10-frontend.git'
        FRONTEND_BRANCH = 'main'
        FRONTEND_DIR = 'ors10-frontend'
        FRONTEND_DIST = 'ors10-frontend\\dist\\p10-ui'

        TOMCAT_DIR = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1\\webapps\\ORS"
        CATALINA_HOME = "C:\\Program Files\\Apache Software Foundation\\Tomcat 10.1"
    }

    stages {
        stage('Checkout Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    deleteDir()
                    git branch: "${FRONTEND_BRANCH}", url: "${FRONTEND_REPO}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo "📦 Installing npm packages..."
                    bat "${NODE_HOME}\\npm install --legacy-peer-deps"
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir("${FRONTEND_DIR}") {
                    script {
                        echo "🏗 Trying normal Angular build..."
                        def status = bat(
                            script: """
                                "${NODE_HOME}\\npx" ng build --configuration production --base-href /ORS/
                            """,
                            returnStatus: true
                        )

                        if (status != 0) {
                            echo "⚠️ Normal build failed, trying fallback build (AOT/optimizer disabled)..."
                            bat """
                                "${NODE_HOME}\\npx" ng build --configuration production --base-href /ORS/ --aot=false --build-optimizer=false
                            """
                        } else {
                            echo "✅ Normal build successful."
                        }
                    }
                }
            }
        }

        stage('Clean Tomcat ORS Folder') {
            steps {
                echo "🧹 Cleaning Tomcat ORS folder..."
                bat "rmdir /S /Q \"${TOMCAT_DIR}\" || echo Folder not found, skipping..."
                bat "mkdir \"${TOMCAT_DIR}\""
            }
        }

        stage('Deploy Frontend') {
            steps {
                echo "🚀 Deploying frontend..."
                bat "xcopy \"${FRONTEND_DIST}\" \"${TOMCAT_DIR}\" /E /H /Y /I"
            }
        }

        stage('Start Tomcat') {
            steps {
                echo "🚀 Starting Tomcat server..."
                script {
                    def status = bat(
                        script: """
                            set CATALINA_HOME=${CATALINA_HOME}
                            set JAVA_HOME=${JAVA_HOME}
                            %CATALINA_HOME%\\bin\\startup.bat
                        """,
                        returnStatus: true
                    )
                    if (status != 0) {
                        error("❌ Tomcat failed to start. Check manually.")
                    } else {
                        echo "✅ Tomcat started successfully."
                    }
                }
            }
        }
    }

    post {
        always {
            echo "✅ Frontend pipeline completed."
        }
    }
}
