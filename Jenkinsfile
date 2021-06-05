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
                        sh 'ls'
                    }
                }

            }
        }
        stage('Development deploy approval and deployment') {
            steps {
                script {
                    if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                        sh 'ls'
                             sh 'kill -9 $(lsof -t -i:8082) || echo "Process was not running."'
                             sh 'echo "mvn spring-boot:run" | at now + 1 minutes'
                        }
                    }
                }
            }
        }

// The options directive is for configuration that applies to the whole job.
    options {
        // For example, we'd like to make sure we only keep 10 builds at a time, so
        // we don't fill up our storage!
        buildDiscarder(logRotator(numToKeepStr: '5'))

        // And we'd really like to be sure that this build doesn't hang forever, so
        // let's time it out after an hour.
        timeout(time: 25, unit: 'MINUTES')
    }
}
