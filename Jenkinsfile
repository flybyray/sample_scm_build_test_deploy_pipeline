#!/usr/bin/env groovy

// only 20 builds
properties([buildDiscarder(logRotator(artifactNumToKeepStr: '20', numToKeepStr: '20'))])

node {
    def dockerTool = tool name: 'Default', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
    stage('checkout and prepare build environment') {
        checkout([$class                           : 'GitSCM',
                  branches                         : [[name: '*/master']],
                  doGenerateSubmoduleConfigurations: false,
                  extensions                       : [[$class: 'CleanCheckout']],
                  submoduleCfg                     : [],
                  userRemoteConfigs                : [[url: 'https://github.com/flybyray/sample_scm_build_test_deploy_pipeline.git']]])
        withCredentials([[$class       : 'StringBinding',
                          credentialsId: 'ndgitSecret',
                          variable     : 'SECRET']]) {
            withEnv(["PATH+DOCKER=${dockerTool}/bin"]) {
                sh 'docker build -t ndgit_build_env --build-arg GID=$(id -g ${USER}) --build-arg UID=$(id -u ${USER}) --build-arg SECRET=${SECRET} - < Dockerfile.build'
            }
        }
    }
    stage('build and ') {
        configFileProvider([configFile(fileId: 'ndgit-maven-settings', targetLocation: 'settings.xml')]) {
            docker.image('ndgit_build_env', 'Default').inside {
                withEnv(['GIT_AUTHOR_NAME=Jenkins', 'GIT_AUTHOR_EMAIL=jenkins@ndgit.com']) {
                    try {
                        sh "mvn clean install -s settings.xml"
                        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/*.xml'])
                        step([$class: 'ArtifactArchiver', artifacts: '*/target/*.jar'])

                        triggerAcceptanceTest()
                    } catch (err) {
                        currentBuild.result = "FAILURE"

                        if (err.toString().contains('AbortException')) {
                            currentBuild.result = "ABORTED"
                        }
                    } finally {
                        sendMail()
                    }
                }
            }
        }
    }
}

def triggerAcceptanceTest() {
    // bundle artifacts for quick access

    try {
        echo "Will attempt to run acceptance tests with the same branch name i.e. '${env.BRANCH_NAME}'."
        /*build(job: "AcceptanceTest-Jenkinsfile/${env.BRANCH_NAME}",
                parameters: [string(name: 'AT_BRANCH_NAME', value: "${env.BRANCH_NAME}"),
                             string(name: 'BUILD_NUM', value: "${env.BUILD_NUMBER}")],
                wait: false)*/
    } catch (ignored) {
        echo "Failed to run the acceptance tests with the same branch name i.e. '${env.BRANCH_NAME}'. Will try running the acceptance tests 'master' branch."
        /*build(job: "AcceptanceTest-Jenkinsfile/master",
                parameters: [string(name: 'AT_BRANCH_NAME', value: "${env.BRANCH_NAME}"),
                             string(name: 'BUILD_NUM', value: "${env.BUILD_NUMBER}")],
                wait: false)*/
    }
}

def sendMail() {
    res = currentBuild.result
    if (currentBuild.result == null) {
        res = "SUCCESS"
    }
    message = "${env.JOB_NAME} #${env.BUILD_NUMBER}, status: ${res} (<a href='${env.RUN_DISPLAY_URL}'>Open</a>)"

    mail body: message,
            from: 'jenkins@ndgit.com',
            replyTo: 'jenkins@ndgit.com',
            subject: "${env.JOB_NAME} #${env.BUILD_NUMBER}, status: ${res}",
            to: 'jenkins@ndgit.com'
}
