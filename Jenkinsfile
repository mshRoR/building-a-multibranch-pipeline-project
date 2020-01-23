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
        stage('Build') {
            steps {
                echo 'Building... dev'
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
        stage('Deliver for master branch') {
            when {
                branch 'master' 
            }
            steps {
                sh './jenkins/scripts/deliver-for-development.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}
