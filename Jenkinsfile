#!groovy

import groovy.json.JsonSlurper
import java.net.URL

pipeline {
    agent none
    options {
        timeout(time: 1, unit: 'DAYS')
        disableConcurrentBuilds()
    }
    stages {
        stage("Init") {
            agent any
            steps { initialize() }
        }
        stage("Build App") {
            agent any 
            steps { buildApp() }
        }
        
    }
}


// ================================================================================================
// Initialization steps
// ================================================================================================

def initialize() {
    env.SYSTEM_NAME = "DSO"
    env.AWS_REGION = "us-east-1"
    env.REGISTRY_URL = "https://912661153448.dkr.ecr.us-east-1.amazonaws.com"
    env.MAX_ENVIRONMENTNAME_LENGTH = 32
    setEnvironment()
    env.IMAGE_NAME = "hello-world:" + 
        ((env.BRANCH_NAME == "master") ? "" : "${env.ENVIRONMENT}-") + 
        env.BUILD_ID
    showEnvironmentVariables()
}

def setEnvironment() {
    def branchName = env.BRANCH_NAME.toLowerCase()
    def environment = 'dev'
    echo "branchName = ${branchName}"
    if (branchName == "") {
        showEnvironmentVariables()
        throw "BRANCH_NAME is not an environment variable or is empty"
    } else if (branchName != "master") {
        //echo "split"
        if (branchName.contains("/")) {
            // ignore branch type
            branchName = branchName.split("/")[1]
        }
        //echo "remove '-' characters'"
        branchName = branchName.replace("-", "")
        //echo "remove JIRA project name"
        if (env.JIRA_PROJECT_NAME) {
            branchName = branchName.replace(env.JIRA_PROJECT_NAME, "")
        }
        // echo "limit length"
        branchName = branchName.take(env.MAX_ENVIRONMENTNAME_LENGTH as Integer)
        environment += "-" + branchName
    }
    echo "Using environment: ${environment}"
    env.ENVIRONMENT = environment
}

def showEnvironmentVariables() {
    sh 'env | sort > env.txt'
    sh 'cat env.txt'
}

// ================================================================================================
// Build steps
// ================================================================================================

def buildApp() {
     dir("webapp") {
        withDockerContainer("maven:3.5.0-jdk-8-alpine") { sh "mvn clean install"}
        archiveArtifacts '**/target/spring-boot-web-jsp-1.0.war'
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'] )
     }
}

