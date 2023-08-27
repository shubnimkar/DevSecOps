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
              sh 'docker run --rm -v /var/lib/jenkins/workspace/DevSecOps:/trufflehog/trufflehog-output trufflesecurity/trufflehog:latest github --org=trufflesecurity -format json -output /trufflehog/trufflehog-output/truffelhog_output.json
'
              sh 'trufflehog3 -R report.json --output report.html'
      }
         
    }
}
}

