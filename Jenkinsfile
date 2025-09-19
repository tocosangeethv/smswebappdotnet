pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = "C:\\Program Files\\dotnet"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Restoring dependencies
                    //bat "cd ${DOTNET_CLI_HOME} && dotnet restore"
                    bat "dotnet restore"

                    // Building the application
                    bat "dotnet build --configuration Release"
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Running tests
                    bat "dotnet test --no-restore --configuration Release"
                }
            }
        }

        stage('Publish') {
            steps {
                script {
                    // Publishing the application
                    bat "dotnet publish --no-restore --configuration Release --output .\\publish"
                }
            }
        }
    

    stage('Deploy to App Server') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'app-server-creds',
                                                 usernameVariable: 'APP_USER',
                                                 passwordVariable: 'APP_PASS')]) {
                    script {
                        def remoteServer    = "20.40.52.13"
                        // Escape the $ in c$  ->  c\\$
                        def destinationPath = "\\\\${remoteServer}\\c\\\$\\inetpub\\wwwroot\\MyApp"
                        def localPath       = "${WORKSPACE}\\publish\\*"

                        // Map the admin share with credentials
                        bat "net use \"${destinationPath}\" /user:%APP_USER% %APP_PASS%"

                        // Copy the published bits
                        bat """
                        powershell -NoProfile -ExecutionPolicy Bypass -Command ^
                          "Copy-Item -Path '${localPath}' -Destination '${destinationPath}' -Recurse -Force"
                        """

                        // Clean up the mapping
                        bat "net use \"${destinationPath}\" /delete"
                    }
                }
            }
    }
}

    post {
        success {
            echo 'Build, test, and publish successful!'
        }
    }
}




