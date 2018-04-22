#!/usr/bin/env groovy

node {
    try {
        notifyBuild('STARTED')

        stage('Init') {
            env.PATH="${tool 'Node 7.x'}/bin:./node_modules/.bin:${env.PATH}"
            // echo sh(script: 'env|sort', returnStdout: true) // will print secret env vars, only use for debug
        }
        stage('checkout') {
            deleteDir()
            checkout scm
        }
        stage('Install') {
            sh 'yarn --version'
            sh 'yarn install'
        }
        stage ('test') {
            sh "yarn test"
        }

        if (env.BRANCH_NAME == "master") {
            stage('package') {
                sh "yarn package"
                sh "yarn package-win"
            }

            post {
                always {
                    archiveArtifacts artifacts: 'release/**/*.dmg', fingerprint: true
                    archiveArtifacts artifacts: 'release/**/*.zip', fingerprint: true
                }
            }
        }


    } catch (e) {
        currentBuild.result = "FAILED"
        throw e
    } finally {
        notifyBuild(currentBuild.result)
    }
}

def notifyBuild(String buildStatus = 'STARTED') {

    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESSFUL'

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
    def summary = "${subject} (${env.BUILD_URL})"
    def details = """<p>STARTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
    <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>"""

    // Override default values based on build status
    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESSFUL') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    // Send notifications
//    slackSend(color: colorCode, message: summary)

    emailext(
        subject: subject,
        body: details,
        recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
}
