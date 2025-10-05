pipeline {
    agent any
    
    tools {
        jdk 'JAVA_HOME'       
        maven 'MAVEN_HOME'     
    }
    
    environment {
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
                    // Use dynamic path that works for any user
                    def warFilePath = "${env.WORKSPACE}\\target\\roshambo.war"
                    echo "Looking for WAR file at: ${warFilePath}"

                    if (fileExists(warFilePath)) {
                        echo '✅ WAR file found, deploying...'
                        
                        withCredentials([usernamePassword(credentialsId: 'tomcat-creds',
                                                          usernameVariable: 'TOMCAT_USER',
                                                          passwordVariable: 'TOMCAT_PASSWORD')]) {
                            bat """
                                curl --upload-file "${warFilePath}" ^
                                --user %TOMCAT_USER%:%TOMCAT_PASSWORD% ^
                                "${TOMCAT_URL}/manager/text/deploy?path=/roshambo&update=true"
                            """
                        }
                    } else {
                        echo '❌ WAR file not found! Checking target directory:'
                        bat "dir \"${env.WORKSPACE}\\target\""
                        error('WAR file not found! Check the directory listing above.')
                    }
                }
            }
        }
    } // end stages

    post {
        always {
            echo 'Build completed'
        }
        success {
            archiveArtifacts 'target/*.war'
        }
    }
} // end pipeline
