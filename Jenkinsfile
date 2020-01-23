#!/usr/bin/env groovy

pipeline {
    agent any
    options { skipDefaultCheckout() }
    // stages {
    //     stage('Build') {
    //         steps {
    //             sh 'echo "webhook for development branch"'
    //             sh 'echo "generic webhook testing using pipeline project"'
    //         }
    //     }
    // }
    stages {
        stage('Deliver for master branch') {
            when {
                branch 'master' 
            }
            stage('Build') {
                steps {
                    echo 'Building...'
                }
            }
            stage('Test') {
                steps {
                    echo 'Testing...'
                }
            }
            stage('Deploy') {
                steps {
                    echo 'Deploying...'
                }
            }
        }
    }
}
