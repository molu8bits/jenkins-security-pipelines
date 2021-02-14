pipeline {
  agent {
    kubernetes {
      yamlFile './podTemplates/defaultPod.yaml'
      defaultContainer 'base-buildserver'
    }
  }
  parameters {
    string defaultValue: 'quay.io/molu8bits/someimage', description: 'image to scan, without the tag', name: 'imageToScan', trim: true
    string defaultValue: 'http://anchore-internal-service-url:8228/v1', description: 'Anchore API Url', name: 'engineUrl', trim: true
    string defaultValue: 'latest', description: "TAG Image. When empty the script will obtain the newest image tag", name: 'tagToScan', trim: true
    booleanParam defaultValue: false, description: 'Force analyze (discard the cache)', name: 'forceAnalyze'
    credentials(name: 'anchoreCredentials', description: 'Anchore Engine Credentials', defaultValue: 'anchore-engine-api', credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', required: true)
    credentials(name: 'artifactoryCredentials', description: 'Artifactory Credentials to find newest tag', defaultValue: 'jenkins-jfrogartifactory-creds', credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl', required: false)
  }
  environment {
    somevalue = 'doesntmatter'
  }
  stages {
    stage('Scan prepare') {
      when {
          expression { params.tagToScan == '' }
      }
      steps {
        println "Scan prepare"
        withCredentials([usernamePassword(credentialsId: "${artifactoryCredentials}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          script {
            repoUrl = params.imageToScan.split('/')[0]
            def imagePath = params.imageToScan.split('/')[1..-1].join('/')
            def artifactoryApiUrl = 'https://my-jfrog-repository-url/artifactory/api/docker/my-repo-name/v2/' + imagePath + "/tags/list?"
            println "artifactoryApiUrl:=${artifactoryApiUrl}"
            tagToScan = sh (
              script: "curl -s -u ${USERNAME}:\${PASSWORD} -XGET ${artifactoryApiUrl} | jq -r .tags[-1]",
              returnStdout: true
            ).trim()
            println "tagToScan:=${tagToScan}"
          }
        }
      }
    }
    stage('Scan execute') {
      steps {
        sh 'echo "Scanning image"'
        script {
          def imageLine = "${imageToScan}:${tagToScan}"
          def buildUser = ''
          wrap([$class: 'BuildUser']) {
            buildUser = "${env.BUILD_USER}"
          }
          currentBuild.description = "${buildUser}: " + "${imageLine}"
          println "imageLine:=${imageLine}"
          writeFile file: 'anchore_images', text: imageLine
          anchore bailOnFail: false, engineCredentialsId: "${anchoreCredentials}", engineurl: "${engineUrl}", forceAnalyze: params.forceAnalyze, name: "anchore_images"
        }
      }
    }
    stage('Collect reports') {
      steps {
        script {
          echo "To be implemented ..."
        }
      }
    }
  }
}