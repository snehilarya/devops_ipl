pipeline {
    // run on jenkins nodes tha has java 8 label
    agent any
    // global env variables
    environment {
        EMAIL_RECIPIENTS = 'snehilarya1@gmail.com'
    }
    stages {

        stage('Build') {
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
        stage('Development deploy') {
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
        stage('Sonar scan execution') {
            // Run the sonar scan
            steps {
                script {
                    def mvnHome = tool 'Jenkins-maven'
                    withSonarQubeEnv {
                        sh "'${mvnHome}/bin/mvn'  verify sonar:sonar -Dintegration-tests.skip=true -Dmaven.test.failure.ignore=true"
                    }
                }
            }
        }

    }
    post {
        // Always runs. And it runs before any of the other post conditions.
        success {
            sendEmail("Successful");
        }
        unstable {
            sendEmail("Unstable");
        }
        failure {
            sendEmail("Failed");
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
def developmentArtifactVersion = ''
def releasedVersion = ''
// get change log to be send over the mail
@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""

    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }

    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}

def sendEmail(status) {
    mail(
            to: "$EMAIL_RECIPIENTS",
            subject: "Build $BUILD_NUMBER - " + status + " (${currentBuild.fullDisplayName})",
            body: "Changes:\n " + getChangeString() + "\n\n Check console output at: $BUILD_URL/console" + "\n")
}
