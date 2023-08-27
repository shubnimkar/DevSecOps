pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git changelog: false, poll: false, url: 'https://github.com/shubnimkar/CI_CD_Devsecops.git'
            }
        }

        stage ('Check secrets') {
          steps {
              sh 'trufflehog3 https://github.com/shubnimkar/CI_CD_Devsecops.git --json /var/lib/jenkins/workspace/DevSecOps > trufflehog_report.json || true'
              sh 'cd /var/lib/jenkins/workspace/DevSecOps'
              sh './truffelhog_report.sh'
      }
          /*  post {
        always {
            // Publish the Trufflehog HTML report using the "Publish HTML reports" plugin
            publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: '',
                reportFiles: 'truffelhog_report.html',
                reportName: 'Trufflehog Scan Report'
            ])
        }
    }*/
        
    }
}
}

