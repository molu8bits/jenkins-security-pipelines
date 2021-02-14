pipeline {
  agent {
    kubernetes {
      yamlFile './podTemplates/podDependencyCheck.yaml'
      defaultContainer 'depcheck'
    }
  }
  options {
    timestamps()
    ansiColor("xterm")
    skipDefaultCheckout(true)
    disableConcurrentBuilds()
  }
  stages {
    stage("Prepare Env Dependency Check") {
      steps {
        sh 'printf "Creating data folder to process files /tmp/depcheckdata"'
        sh 'mkdir /tmp/depcheckdata'
      }
    }
    stage('Update Dependency Check DB') {
        environment {
            depcheckConnectionString = 'jdbc:mysql://mariadb.url/dependencycheck'
            depcheckDriverPath = '/usr/share/dependency-check/plugins/mysql-connector-java-8.0.17.jar'
            depcheckDriver = 'com.mysql.cj.jdbc.Driver'
        }
        steps {
            withCredentials([usernamePassword(credentialsId: 'jenkins-mariadb-depcheckdb-creds', usernameVariable: 'DEPUSERNAME', passwordVariable: 'DEPPASSWORD')]) {
                script {
            sh 'echo "depcheckConnectionString:=\$depcheckConnectionString"'
            # Version with proxy server
            # sh '/usr/share/dependency-check/bin/dependency-check.sh --dbDriverName \$depcheckDriver --connectionString \$depcheckConnectionString --dbUser \$DEPUSERNAME --dbPassword \$DEPPASSWORD --dbDriverPath \$depcheckDriverPath --updateonly --proxyserver myproxyserver --proxyport 8080 -d /tmp/depcheckdata'
            # Version without proxy server
            sh '/usr/share/dependency-check/bin/dependency-check.sh --dbDriverName \$depcheckDriver --connectionString \$depcheckConnectionString --dbUser \$DEPUSERNAME --dbPassword \$DEPPASSWORD --dbDriverPath \$depcheckDriverPath --updateonly --proxyserver myproxyserver --proxyport 8080 -d /tmp/depcheckdata'
          }
        }
      }
    }
    stage('Get Dependency Check logs') {
      steps {
        containerLog 'depcheck'
      }
    }
  }
}