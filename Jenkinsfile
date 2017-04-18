#!/usr/bin/env groovy

// only 20 builds
properties([buildDiscarder(logRotator(artifactNumToKeepStr: '20', numToKeepStr: '20'))])

pipeline {
    agent any
    stages {
        stage('prepare build image') {
            steps {
                withCredentials([[$class       : 'StringBinding',
                                  credentialsId: 'ndgitSecret',
                                  variable     : 'SECRET']]) {
                    sh 'docker build -t gs_build_env --build-arg GID=$(id -g ${USER}) --build-arg UID=$(id -u ${USER}) --build-arg SECRET=${SECRET} - < Dockerfile.build'
                }
            }
        }
    }
}