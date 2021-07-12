#!/usr/bin/env groovy

pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timeout(time: 1, unit: 'HOURS')
        timestamps()
    }

    environment {
        //POM_VERSION = getVersion()
        //JAR_NAME = getJarName()
        
        VERSION = 'latest'
        PROJECT = 'test-repo'
        IMAGE = 'test-repo:latest'
        ECRURL = 'http://582495273871.dkr.ecr.ap-south-1.amazonaws.com'
        ECRCRED = 'ecr:ap-south-1:eb87d016-ca68-4b56-89af-aca41c31faaf'  
        AWS_ECR_REGION = 'ap-south-1'
        AWS_ECS_SERVICE = 'ch-dev-user-api-service'
        AWS_ECS_TASK_DEFINITION = 'ch-dev-user-api-taskdefinition'
        AWS_ECS_COMPATIBILITY = 'FARGATE'
        AWS_ECS_NETWORK_MODE = 'awsvpc'
        AWS_ECS_CPU = '256'
        AWS_ECS_MEMORY = '512'
        AWS_ECS_CLUSTER = 'ch-dev'
        AWS_ECS_TASK_DEFINITION_PATH = './ecs/container-definition-update-image.json'
    }

    stages {
        stage('Build preparations') {
            steps
            {
                script 
                {
                    // calculate GIT lastest commit short-hash
                    gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    shortCommitHash = gitCommitHash.take(7)
                    // calculate a sample version tag
                    VERSION = shortCommitHash
                    // set the build display name
                    currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
                    IMAGE = "$PROJECT:$VERSION"
                }
            }
        }
        stage('Build Docker Image') {
              steps {
                withCredentials([string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
                  script {
                    docker.build("${AWS_ECR_URL}:${POM_VERSION}", "--build-arg JAR_FILE=${JAR_NAME} .")
                  }
                }
              }
        }
        
        stage('Docker build')
        {
            steps
            {
                script
                {
                    // Build the docker image using a Dockerfile
                    docker.build("$IMAGE")
                }
            }
        }
        stage('Docker push')
        {
            steps
            {
                script
                {
                    // login to ECR - for now it seems that that the ECR Jenkins plugin is not performing the login as expected. I hope it will in the future.
                    sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
                    // Push the Docker image to ECR
                    docker.withRegistry(ECRURL, ECRCRED)
                    {
                        docker.image(IMAGE).push()
                    }
                }
            }
        }  
        

        //stage('Deploy in ECS') {}
    }

    post
    {
        always
        {
            // make sure that the Docker image is removed
            sh "docker rmi $IMAGE | true"
        }
    }
}
