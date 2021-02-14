pipeline {
  agent {
    kubernetes {
      yamlFile './podTemplates/dindPod.yaml'
      defaultContainer 'dind'
    }
  }
  parameters {
    string defaultValue: '', description: 'Repository with Dockerfile to build. E.g github.com/myusername/myproject. When empty runs in DEMO mode', name: "repositoryWithDockerfile", trim: true
    string defaultValue: './', description: 'Location of the Dockerfile inside repository. Usually defaults to "./".', name: 'dockerfileDirectory', trim: true
    booleanParam defaultValue: false, description: 'Use git credentials to checkout Dockerfile repo', name: 'scmUseCreds'
    credentials(name: 'scmCredentials', description: 'SCM credentials to get repo with Dockerfile', defaultValue: 'jenkins-scm-creds', credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', required: false)
    string defaultValue: 'quay.io/molu8bits/test-image-docker', description: 'Name of the image to build and publish', name: "registry", trim: true
    string defaultValue: '', description: 'Tag of the image to publish. When empty then Jenkins BUILD_NUMBER will be used', name: "registryTag", trim: true
    credentials(name: 'registryCredentials', description: 'Docker registry credentials to authenticate and publish images', defaultValue: 'jenkins-repository-creds', credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', required: false)
    booleanParam defaultValue: false, description: 'Test image. Currently only runs echo command inside freshly built image', name: 'testImage'
    booleanParam defaultValue: true, description: 'Publish image to Docker repository', name: 'publishImage'
  }
  environment {
    somevalue = 'someparam'
  }
  stages {
    stage('Clone DEMO repository') {
      when {
        expression { params.repositoryWithDockerfile == "" }
      }
      steps {
        script {
          println "Debug information"
          println "registry:=${registry}"
          println "BUILD_NUMBER:=${BUILD_NUMBER}"
          println "TEST_IMAGE:=${params.TEST_IMAGE}"
          sh 'echo PWD:=\${PWD}'
          sh 'echo "\$DOCKER_HOST"'
          sh 'docker version'
          sh 'echo "ls -ltr"'
          sh 'ls -ltr'
          sh 'find . -name Dockerfile'
          sh 'whoami'
        }
        checkout scm
      }
    }
    stage('Clone user repository') {
      when {
        expression { params.repositoryWithDockerfile != "" }
      }
      steps {
        script {
          if ( params.scmUseCreds == true) {
            checkout([
              $class: 'GitSCM', branches: [[name: '*/master']], 
              doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], 
              userRemoteConfigs: [[credentialsId: "${scmCredentials}", 
              url: "${repositoryWithDockerfile}"]]]
            )
          }
          else {
            git url: "${repositoryWithDockerfile}"
            println "----- DEBUG Information -----"
            sh "ls -ltr"
            sh "whoami"
            sh "pwd"
            println "----- ----- ----------- -----"
          }
        }
      }
    }
    stage('Build Image') {
      steps {
        sh 'echo "Building image"'
        script {
          dockerImage = "${registry}:latest"
          sh """
            docker build -t "\${registry}" "\${dockerfileDirectory}"
          """
        }
      }
    }
    stage('Test image') {
      when {
        expression { params.testImage == true }
      }
      steps {
        script {
          docker.image("${registry}").withRun("bash -c") {
            echo "Hello from the Container Side"
          }
        }
      }
    }
    stage('Publish Image') {
      when {
        expression { params.publishImage == true }
      }
      steps {
        println "Publishing image"
        withDockerRegistry([ credentialsId: params.registryCredentials, url: "https://${registry}"]) {
          script {
            println "registry:=${registry}"
            println "BUILD_NUMBER:=${BUILD_NUMBER}"
            if ( registryTag == '' ) {
                println "Tagging with BUILD_NUMER=:${BUILD_NUMBER}"
                sh """
                  docker tag "${registry}" "${registry}:${BUILD_NUMBER}"
                  docker push "${registry}:${BUILD_NUMBER}"
                """
            }
            else {
                println "Tagging with param.registryTag=:${registryTag}"
                sh """
                  docker tag "${registry}" "${registry}:${registryTag}"
                  docker push "${registry}:${registryTag}"
                """
            }
            sh """
              docker push \"${registry}:latest"
            """
          }
        }
      }
    }
    stage('Cleanup unused docker image') {
      steps {
        catchError(buildResult: 'SUCCESS', stageResult: 'ABORTED') {
          script {
            println 'Cleaning up the node space'
            if ( registryTag == '' ) {
              sh "docker rmi ${registry} ${registry}:${BUILD_NUMBER}"
            }
            else {
              sh "docker rmi ${registry} ${registry}:${registryTag}"
            }
          }
        }
      }
    }
  }
}
