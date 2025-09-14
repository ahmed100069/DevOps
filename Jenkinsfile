pipeline {
    agent any
    
    tools {
        jdk 'JAVA_HOME'
        maven 'MAVEN_HOME'
    }
    
    environment {
        // Remove hardcoded WAR_FILE path
        TOMCAT_URL = 'http://localhost:7080'
    }
    
    stages {
        stage('Clean Project') {
            steps {
                bat "mvn clean"
            }
        }

        stage('Build Project') {
            steps {
                bat "mvn package"
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    // Automatically locate the WAR file inside the 'web/target' directory
                    def warFile = findFiles(glob: 'web/target/*.war')[0]
                    echo "WAR file found at: ${warFile.path}"

                    if (warFile) {
                        echo 'Deploying WAR to Tomcat...'

                        withCredentials([usernamePassword(credentialsId: 'tomcat-creds',
                                                         usernameVariable: 'TOMCAT_USER',
                                                         passwordVariable: 'TOMCAT_PASSWORD')]) {
                            bat """
                                curl --upload-file "${warFile.path}" ^
                                --user %TOMCAT_USER%:%TOMCAT_PASSWORD% ^
                                "${TOMCAT_URL}/manager/text/deploy?path=/roshambo&update=true"
                            """
                        }
                    } else {
                        error('WAR file not found!')
                    }
                }
            }
        }
    } // end stages

    post {
        always {
            echo 'Build completed'
        }
    }
} // end pipeline
