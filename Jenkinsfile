#!/usr/bin/env groovy

// only 20 builds
properties([buildDiscarder(logRotator(artifactNumToKeepStr: '20', numToKeepStr: '20'))])

pipeline {
    agent any
    node {
        def dockerTool = tool name: 'Default', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        stages {
            stage('checkout and prepare build environment') {
                steps {
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
                            sh 'docker build -t gs_build_env --build-arg GID=$(id -g ${USER}) --build-arg UID=$(id -u ${USER}) --build-arg SECRET=${SECRET} - < Dockerfile.build'
                        }
                    }
                }
            }
        }
    }
}