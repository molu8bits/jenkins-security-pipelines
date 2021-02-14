pipeline {
  agent {
    kubernetes {
      yamlFile './podTemplates/podScoutSuite.yaml'
      defaultContainer 'scoutsuite'
    }
  }
  parameters {
      choice(name: "testexecute", choices: ['True','False','I dont know'], description: 'True to execute AWS security test')
      credentials(name: 'AWS_CREDENTIALS', description: 'Pick up defined AWS credentials', defaultValue: '', credentialType: 'AWS Credentials', required: true)
  }
  options {
      timestamps()
      ansiColor("xterm")
      skipDefaultCheckout(true)
  }
  stages {
    stage("Check AWS Credentials") {
      steps {
        sh 'echo "Show current AWS user credentials"'
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AWS_CREDENTIALS
          //accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          container('awscli') {
            sh 'aws sts get-caller-identity'
          }
        }
      }
    }
    stage('AWS ScoutSuite Scan') {
      when {
        environment name: "testexecute", value: "True"
      }
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding', credentialsId: env.AWS_CREDENTIALS
        ]]) {
            sh 'printf "\\e[35mScoutSuite AWS in progress...\\e[0m\\n"'
            sh 'python /opt/scout.py aws'
          }
        }
    }
    stage('Publish report') {
      when {
        environment name: "testexecute", value: "True"
      }
      steps {
        script {
          sh 'ls scoutsuite-report/*html > reportFilename.txt'
          sh 'cat reportFilename.txt'
          def reportFilename = readFile('reportFilename.txt').split("/")[1]
          println "reportFilename:=${reportFilename}"
          publishHTML (target: [
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: true,
            reportDir: "scoutsuite-report",
            reportFiles: "${reportFilename}",
            includes: "**/**",
            reportName: "ScoutSuite AWS Report"
          ])
        }
      }
    }
    stage('Archive artifacts') {
      when {
        environment name: "testexecute", value: "True"
      }
      steps {
        zip zipFile: 'scoutsuite-report.zip', archive: false, dir: 'scoutsuite-report'
        archiveArtifacts artifacts: "scoutsuite-report.zip", fingerprint: true
      }
    }
    stage('Get ScoutSuite container logs') {
      steps {
        containerLog 'scoutsuite'
      }
    }
  }
}