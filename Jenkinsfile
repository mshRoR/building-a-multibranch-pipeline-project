#!/usr/bin/env groovy

pipeline {
    options { buildDiscarder(logRotator(numToKeepStr: '3')) }
    agent any
    environment {
        VERSION = 'latest'
        ECRURL = '703877095045.dkr.ecr.eu-west-1.amazonaws.com'
        REGION = 'eu-west-1'

        ECRREPO = ''
        CLUSTER_NAME = ''
        SERVICE_NAME = ''
        TASK_DEFINTION_NAME = ''
        EMAIL = ''
        BRANCH = ''
    }
    stages {
        stage('Build preparations') {
            steps {
                script {
                    FULL_PATH_BRANCH = sh(script:'git name-rev --name-only HEAD', returnStdout: true)
                    BRANCH = FULL_PATH_BRANCH.substring(FULL_PATH_BRANCH.lastIndexOf('/') + 1, FULL_PATH_BRANCH.length())
                    // git committer email
                    EMAIL = sh(returnStdout: true, script: 'git show -s --pretty=%ae').trim()

                    // calculate GIT latest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(8)

                    // calculate a sample version tag
                    VERSION = shortCommitHash

                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                }
            }
        }
        stage('Development Server Stage') {
            when {
                expression { GIT_BRANCH == 'origin/development' }
            }
            steps {
                script {
                    ECRREPO = 'dev-finance-frontend'
                    CLUSTER_NAME = 'dev-finance-cluster'
                    SERVICE_NAME = 'dev-finance-frontend-service'
                    TASK_DEFINTION_NAME = 'dev-finance-frontend'
                    echo "========DEVELOPMENT BRANCH=========="
                    echo "MY BRANCH: $BRANCH"
                    echo "GIT_BRANCH: $GIT_BRANCH"
                }
            }
        }
        stage('Production Server Stage') {
            when {
                expression { GIT_BRANCH == 'origin/production' }
            }
            steps {
                script {
                    ECRREPO = 'prod-finance-frontend'
                    CLUSTER_NAME = 'prod-finance-cluster'
                    SERVICE_NAME = 'prod-finance-frontend-service'
                    TASK_DEFINTION_NAME = 'prod-finance-frontend'
                }
            }
        }
        stage('Testing Stage') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            steps {
                script {
                    ECRREPO = 'jenkins-test'
                    CLUSTER_NAME = 'dev-finance-cluster'
                    SERVICE_NAME = 'dev-finance-frontend-service'
                    TASK_DEFINTION_NAME = 'dev-finance-frontend'
                    
                    sh """
                        echo "EMAIL: $EMAIL"
                        echo "ECRREPO: $ECRREPO"
                        echo "CLUSTER_NAME: $CLUSTER_NAME"
                        echo "SERVICE_NAME: $SERVICE_NAME"
                        echo "TASK_DEFINTION_NAME: $TASK_DEFINTION_NAME"
                        echo "========MASTER BRANCH=========="
                        echo "MY BRANCH: $BRANCH"
                        echo "GIT_BRANCH: $GIT_BRANCH"
                    """
                }
            }
        }
        stage('Docker image building and pushing') {
            when {
                // expression { BRANCH ==~ /(master|development)/ }
                anyOf {
                    expression { BRANCH == 'master' }
                    expression { BRANCH == 'development' }
                    // environment name: 'DEPLOY_TO', value: 'master'
                    // environment name: 'DEPLOY_TO', value: 'development'
                }
            }
            steps {
                script {
                    sh 'printenv'

                    // // login aws ecr
                    // sh "\$(aws ecr get-login --no-include-email --region $REGION)"

                    // // Build the docker image using a Dockerfile
                    // sh """
                    //     echo 'Building docker image ...'
                    //     docker build --build-arg ENV=build-dev -t $ECRURL/$ECRREPO:latest .
                        
                    //     echo 'Tagging image...'
                    //     docker tag $ECRURL/$ECRREPO:latest $ECRURL/$ECRREPO:$VERSION
                        
                    //     echo 'Pushing image to ECR ...'
                    //     docker push $ECRURL/$ECRREPO:latest
                    //     docker push $ECRURL/$ECRREPO:$VERSION
                    // """

                    // // Deploying image to ECS cluster
                    // sh """
                    //     TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition '$TASK_DEFINTION_NAME' --region $REGION)
                    //     NEW_CONTAINER_DEFINTIION=$(echo $TASK_DEFINITION | jq --arg IMAGE '$ECRURL/$ECRREPO' '.taskDefinition.containerDefinitions[0].image = $IMAGE | .taskDefinition.containerDefinitions[0]')
                    //     echo 'Registering new container definition...'
                    //     aws ecs register-task-definition --region '$REGION' --family '$TASK_DEFINTION_NAME' --container-definitions '${NEW_CONTAINER_DEFINTIION}'
                    //     echo 'Check cluster status...'
                    //     aws ecs describe-clusters --region '$REGION' --cluster '$CLUSTER_NAME'
                    //     echo 'Updating the service...'
                    //     aws ecs update-service --region '$REGION' --cluster '$CLUSTER_NAME' --service '$SERVICE_NAME'  --task-definition '$TASK_DEFINTION_NAME' --force-new-deployment
                    // """
                }
            }
        }
    }
    // post {
    //     always {
    //         // make sure that the Docker image is removed
    //         sh """
    //             docker rmi $ECRURL/$ECRREPO:latest | true
    //             docker rmi $ECRURL/$ECRREPO:$VERSION | true
    //         """
    //     }
    //     failure {
    //         mail to:"$EMAIL", subject:"${currentBuild.fullDisplayName}", body:"${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}"
    //     }
    // }
}
