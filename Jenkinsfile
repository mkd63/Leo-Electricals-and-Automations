#!/usr/bin/env groovy

pipeline {
    //parameters
    parameters {
      string defaultValue: 'master', description: '', name: 'BRANCH_NAME', trim: true
    }
    
    agent any
    
    stages {
        stage('Clean workspace') 
        {
            steps {
                cleanWs()
            }
        }

        
        stage('checkout scm'){
            steps{
                git branch: params.BRANCH_NAME, changelog: false, credentialsId: '8da90870-5309-4ae0-8a2f-4a9d201bb38e', poll: false, url: 'https://github.com/mkd63/Leo-Electricals-and-Automations.git'
                script{
                    GIT_COMMIT_SHORT = sh(script: "printf \$(git rev-parse --short HEAD)",
                    returnStdout: true )
                }
            }
        }
        stage('Setting ECR credentials'){
            steps{
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 582495273871.dkr.ecr.ap-south-1.amazonaws.com'
            }
        }
        stage('building image and pushing to ecr'){
            steps{
                sh """#!/bin/bash
                    docker build -t test-repo:latest .
                    docker tag test-repo:latest 582495273871.dkr.ecr.ap-south-1.amazonaws.com/test-repo:latest
                    docker push 582495273871.dkr.ecr.ap-south-1.amazonaws.com/test-repo:latest
                    docker tag test-repo:latest 582495273871.dkr.ecr.ap-south-1.amazonaws.com/test-repo:$GIT_COMMIT_SHORT
                    docker push 582495273871.dkr.ecr.ap-south-1.amazonaws.com/test-repo:$GIT_COMMIT_SHORT
                """
            }
        }  
    }
    post{
        always{
            cleanWs()
            sh 'docker system prune -af'
        }
    }

}
