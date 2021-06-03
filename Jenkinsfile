pipeline {
    // run on jenkins nodes tha has java 8 label
    agent any
    // global env variables
    environment {
        EMAIL_RECIPIENTS = 'snehilarya2@gmail.com'
    }
    stages {

        stage('Build with unit testing') {
            steps {
                // Run the maven build
                script {
                    // Get the Maven tool.
                    // ** NOTE: This 'M3' Maven tool must be configured
                    // **       in the global configuration.
                    echo 'Pulling...' + env.BRANCH_NAME
                    def mvnHome = tool 'Jenkins-maven'
                    if (isUnix()) {
                        sh "'${mvnHome}/bin/mvn' -Dintegration-tests.skip=true clean package"
                        def pom = readMavenPom file: 'pom.xml'
                        // get the current development version
                        print pom.version
                        archive 'target*//*.jar'
                    } else {
                        bat(/"${mvnHome}\bin\mvn" -Dintegration-tests.skip=true clean package/)
                        def pom = readMavenPom file: 'pom.xml'
                        print pom.version
                        archive 'target*//*.jar'
                    }
                }

            }
        }
        stage('Development deploy approval and deployment') {
            steps {
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                                // replace it with your application name or make it easily loaded from pom.xml
                                def jarName = "ipl-dashboard-0.0.1-SNAPSHOT.jar"
                                echo "the application is deploying ${jarName}"
                                // NOTE : CREATE your deployemnt JOB, where it can take parameters whoch is the jar name to fetch from jenkins workspace
                                build job: 'ApplicationToDev', parameters: [[$class: 'StringParameterValue', name: 'jarName', value: jarName]]
                                echo 'the application is deployed !'
                        }
                    }
                }
            }
        }
}