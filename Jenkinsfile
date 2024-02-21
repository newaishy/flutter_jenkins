#!groovy

def lastCommitInfo = ""
def skippingText = ""
def commitContainsSkip = 0
def slackMessage = ""
def shouldBuild = false

def pollSpec = "" 

def appname = "Runner" 
def xcarchive = "${appname}.xcarchive"

if(env.BRANCH_NAME == "master") {
    pollSpec = "*/5 * * * *"
} else if(env.BRANCH_NAME == "test") {
    pollSpec = "* * * * 1-5"
}

pipeline {
    agent any
    // agent { label 'JENKINS_NODE_NAME_HERE' }

    environment {
        DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer/"  //This is necessary for Fastlane to access iOS Build things.
        PATH = "/Users/jenkins/.rbenv/shims:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin:/Users/jenkins/Documents/flutter/bin:/usr/local/Caskroom/android-sdk/4333796//tools:/usr/local/Caskroom/android-sdk/4333796//platform-tools:/Applications/Xcode.app/Contents/Developer"
    }

    options {
        ansiColor("xterm")
    }
 
    triggers {
        pollSCM ignorePostCommitHooks: true, scmpoll_spec: pollSpec
    }

    stages {
        stage('Init') {
            steps {
                script {
                    lastCommitInfo = sh(script: "git log -1", returnStdout: true).trim()
                    commitContainsSkip = sh(script: "git log -1 | grep '.*\\[skip ci\\].*'", returnStatus: true)
                    //If commit string contains [skip ci] we are going to skip the build with setting env.shouldBuild to false.
                    if(commitContainsSkip == 0) {
                        skippingText = "Skipping commit."
                        env.shouldBuild = false
                        currentBuild.result = "NOT_BUILT"
                    }

                    slackMessage = "*${env.JOB_NAME}* *${env.BRANCH_NAME}* received a new commit. ${skippingText}\nHere is commmit info: ${lastCommitInfo}"
                }
            }
        }

        stage('Send info to Slack') {
            steps {
                slackSend color: "#2222FF", message: slackMessage
            }
        }

        stage('Run Unit and UI Tests') {
            when {
                expression {
                    return env.shouldBuild != "false"
                }
            }
            steps {
                script {
                    try {
                        // Run all the tests
                        sh "flutter test --coverage test/logic_tests.dart"
                    } catch(exc) {
                        currentBuild.result = "UNSTABLE"
                        error('There are failed tests.')
                    }
                }
            }
        }

        stage('Build application for beta (Flutter Build APK)') {
            when {
                expression {
                    return env.shouldBuild != "false"
                }
            }
            steps {
                // Steps for beta build
                sh "flutter build apk --split-per-abi"
            }
        }

        stage('Deploy to beta (Distribute Android APK)') {
            when {
                expression {
                    return env.shouldBuild != "false"
                }
            }
            steps {
                // Build steps to deploy application to beta
                appCenter apiToken: 'APITOKENHERE',
                ownerName: 'Phtiv08',
                appName: 'Flutter-Beer',
                pathToApp: 'build/app/outputs/apk/release/app-arm64-v8a-release.apk',
                distributionGroups: 'Flutter-Beer-Distribution'
            }
        }

        stage('Build application for beta (Flutter Build iOS)') {
            when {
                expression {
                    return env.shouldBuild != "false"
                }
            }
            steps {
                // Steps for beta build
                sh "flutter build ios --release --no-codesign"
            }
        }

        stage('Deploy to beta (Make iOS IPA And Distribute)') {
            when {
                expression {
                    return env.shouldBuild != "false"
                }
            }
            steps {
                // Build steps to deploy application to beta
                dir('ios'){
                        sh "bundle install"
                        sh "bundle exec fastlane buildAdHoc --verbose" 
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                sh "flutter clean"
            }
        }
        

        // stage('Build application for prod') {
        //     when {
        //         expression {
        //             return env.shouldBuild != "false" && env.BRANCH_NAME == "master"
        //         }
        //     }
        //     steps {
        //         // Build steps to build application for prod
        //     }
        // }

        // stage('Send to Prod') {
        //     when {
        //         expression {
        //             return env.shouldBuild != "false" && env.BRANCH_NAME == "master"
        //         }
        //     }
        //     steps {
        //         // Steps to deploy application to prod
        //     }
        // }

        stage('Inform Slack for success') {
            when {
                expression {
                    return env.shouldBuild != "false"
                }
            }
            steps {
                slackSend color: "good", message: "*${env.JOB_NAME}* *${env.BRANCH_NAME}* job is completed successfully"
            }
        }
    }

    post {
        failure {
            slackSend color: "danger", message: "*${env.JOB_NAME}* *${env.BRANCH_NAME}* job is failed"
        }
        unstable {
            slackSend color: "danger", message: "*${env.JOB_NAME}* *${env.BRANCH_NAME}* job is unstable. Unstable means test failure, code violation etc."
        }
    }
}