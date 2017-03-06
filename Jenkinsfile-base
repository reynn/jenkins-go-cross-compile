#!/usr/bin/env groovy

// this will start an executor on a Jenkins agent with the docker label
node('docker') {
  // Setup variables
  // application name will be used in a few places so create a variable and use string interpolation to use it where needed
  String applicationName = "basic-app"
  // a basic build number so that when we build and push to Artifactory we will not overwrite our previous builds
  String buildNumber = "0.1.${env.BUILD_NUMBER}"
  // Path we will mount the project to for the Docker container
  String goPath = "/go/src/github.com/reynn/${applicationName}"

  // Checkout the code from Github, stages allow Jenkins to visualize the different sections of your build steps in the UI
  stage('Checkout from GitHub') {
    // No special needs here, if your projects relys on submodules the checkout step would need to be different
    checkout scm
  }

  // Start a docker container using the golang:1.8.0-alpine image, mount the current directory to the goPath we specified earlier
  stage("Create binaries") {
    docker.image("golang:1.8.0-alpine").inside("-v ${pwd()}:${goPath}") {
      for (command in binaryBuildCommands) {
        // build the Mac x64 binary
        sh "cd ${goPath} && GOOS=darwin GOARCH=amd64 go build -o binaries/amd64/${buildNumber}/darwin/${applicationName}-${buildNumber}.darwin.amd64"
        // build the Windows x64 binary
        sh "cd ${goPath} && GOOS=windows GOARCH=amd64 go build -o binaries/amd64/${buildNumber}/windows/${applicationName}-${buildNumber}.windows.amd64.exe"
        // build the Linux x64 binary
        sh "cd ${goPath} && GOOS=linux GOARCH=amd64 go build -o binaries/amd64/${buildNumber}/linux/${applicationName}-${buildNumber}.linux.amd64"
      }
    }
  }

  stage("Archive artifacts") {
    // Archive the binary files in Jenkins so we can retrieve them later should we need to audit them
    archiveArtifacts artifacts: 'binaries/**', fingerprint: true
  }
}
