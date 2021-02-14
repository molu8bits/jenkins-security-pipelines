pipeline {
  agent {
    kubernetes {
      yamlFile './podTemplates/podZap.yaml'
      defaultContainer 'zap2docker'
    }
  }
  parameters {
      string name: 'SCAN_TARGET', defaultValue: 'https://localhost:443/', description: 'Provide URL to scan',  trim: true
      string name: 'reportName', defaultValue: 'ZAP Active Scan report', description: 'Provide Report Name',  trim: true
      string name: 'reportedBy', defaultValue: '', description: 'Provide your name. When empty Jenkins username is used', trim: true

  }
  options {
      timestamps()
      ansiColor("xterm")
      skipDefaultCheckout(true)
  }
  stages {
    stage('Run ZAP in deamon mode') {
      steps {
        sh 'wget --no-verbose https://github.com/zaproxy/zap-extensions/releases/download/exportreport-v6/exportreport-alpha-6.zap -O /zap/plugin/exportreport-alpha-6.zap'
        sh 'zap.sh -daemon -host 127.0.0.1 -newsession "/tmp/session" -config api.disablekey=true -config scanner.attackOnStart=true -config view.mode=attack -config connection.dnsTtlSuccessfulQueries=-1 -config api.addrs.addr.name=.* -config api.addrs.addr.regex=true &'
      }
    }
    stage('Check if ZAP is running') {
      steps {
        sh 'zap-cli status -t 120'
      }
    }
    stage('Run ZAP Basic Scan') {
      steps {
        sh """
          set +e
          zap-cli quick-scan -s all --spider -r \$SCAN_TARGET
          set -e
        """
      }
    }
    stage('Show ZAP alerts') {
      steps {
        sh """
          set +e
          zap-cli alerts
          set -e
        """
      }
    }
    stage('Get ZAP reports') {
      steps {
        sh """
          REPORT_DIR="\$(pwd)/out"
          REPORT_FILE_HTML=report.html
          REPORT_FILE_XML=report.xml
          REPORT_FILE_XHTML=report.xhtml
          REPORT_NAME="\${reportName}"
          echo "reportedBy = \${reportedBy}"
          PREPARED_BY="\${reportedBy}"
          BUILD_VERSION="Scan v.\${BUILD_NUMBER}"
          REPORT_VERSION="Report v.\${BUILD_NUMBER}"
          SCAN_DATE=\$(date +%Y-%m-%d_%H:%M)
          REPORT_DATE=\$(date +%Y-%m-%d_%H:%M)
          DESCRIPTION="Scanned target: \$SCAN_TARGET"
          if [ ! -d "\${REPORT_DIR}" ]; then
            mkdir \${REPORT_DIR} && chmod 777 \${REPORT_DIR}
          fi
          zap-cli -v report --output-format html --output \${REPORT_DIR}/\${REPORT_FILE_HTML}
          zap-cli -v report --output-format xml --output \${REPORT_DIR}/\${REPORT_FILE_XML}
          mkdir /tmp/cpsession
          cp -vr /tmp/session* /tmp/cpsession
          zap.sh -export_report -port 8087 \${REPORT_DIR}/\${REPORT_FILE_XHTML} -source_info "ZAP Baseline scan for App; Reporter Name; Company Name; \${SCAN_DATE}; \${REPORT_DATE}; Scan \$SCAN_VERSION; Report \$REPORT_VERSION; \${DESCRIPTION}" -alert_severity "t;t;f;t" -alert_details "t;t;t;t;t;t;f;f;f;f" -session "/tmp/cpsession/session" -cmd
        """
        archiveArtifacts artifacts: 'out/*', fingerprint: true
      }
    }
    stage('Shutdown ZAP daemon') {
      steps {
        sh 'zap-cli shutdown'
      }
    }
    stage('Get ZAP container logs') {
      steps {
        containerLog 'zap2docker'
      }
    }
  }
}