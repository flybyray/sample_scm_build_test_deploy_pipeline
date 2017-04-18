#!/usr/bin/env groovy

// only 20 builds
properties([buildDiscarder(logRotator(artifactNumToKeepStr: '20', numToKeepStr: '20'))])

pipeline {
    agent any
    stages {
        currentBuild.result = "SUCCESS"
        try {
            stage('Checkout') {
                steps {
                    deleteDir()
                    checkout scm
                    // sh 'git submodule update --init'
                    withCredentials([[$class       : 'StringBinding',
                                      credentialsId: 'ndgitSecret',
                                      variable     : 'SECRET']]) {
                        sh 'docker build -t gs_build_env --build-arg GID=$(id -g ${USER}) --build-arg UID=$(id -u ${USER}) --build-arg SECRET=${SECRET} - < Dockerfile.build'
                    }
                }
            }
        }
        catch (err) {
            currentBuild.result = "FAILURE"
            mail body: "project build error is here: ${env.BUILD_URL}",
                    from: 'jenkins@jenkins.nextbankingapi.com',
                    subject: 'project build failed',
                    to: 'robert@ndgit.com'

            throw err
        }
    }
}