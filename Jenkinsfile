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
    }
}
