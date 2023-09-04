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
        sh 'docker run  gesellix/trufflehog --json https://github.com/shubnimkar/CI_CD_Devsecops.git > trufflehog.json'

        script {
            def jsonReport = readFile('trufflehog.json')
            
            def htmlReport = """
            <html>
            <head>
                <title>Trufflehog Scan Report</title>
            </head>
            <body>
                <h1>Trufflehog Scan Report</h1>
                <pre>${jsonReport}</pre>
            </body>
            </html>
            """
            
            writeFile file: 'scanresults/trufflehog-report.html', text: htmlReport
        }
        
        archiveArtifacts artifacts: 'scanresults/trufflehog-report.html', allowEmptyArchive: true
    }
}

        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml' 
            }
        }

      stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''
    
                }
            }
        }

        stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
        
        stage("Docker Build"){
            steps{
                script{
                   withDockerRegistry([url:'https://index.docker.io/v1/',credentialsId: '69fb7f6f-90ba-4bab-baf3-765387680986', toolName: 'docker']) {
                        sh "docker build -t petclinic1 ."
                    }
                }
            }
        }

	stage("TRIVY") {
   	 steps {
        script {
            def trivyScanOutput = sh(script: 'trivy image shubnimkar/pet-clinic25:latest', returnStatus: true)
            
            if (trivyScanOutput == 0) {
                def htmlReport = sh(script: 'trivy image shubnimkar/pet-clinic25:latest | awk \'{print "<tr><td>" $1 "</td><td>" $2 "</td><td>" $3 "</td></tr>"}\' > trivy-report.html', returnStdout: true)
                
                archiveArtifacts artifacts: 'trivy-report.html', allowEmptyArchive: true
                
                // Publish HTML report using Jenkins' HTML Publisher Plugin
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: '', reportFiles: 'trivy-report.html', reportName: 'Trivy Scan Report'])
            } else {
                error "Trivy scan failed"
            }
        }
    }
}
        
        stage("Docker Tag & Push"){
            steps{
                script{
                   withDockerRegistry([url:'https://index.docker.io/v1/',credentialsId: '69fb7f6f-90ba-4bab-baf3-765387680986', toolName: 'docker']) {
                        sh "docker tag petclinic1 shubnimkar/pet-clinic25:latest"
                        sh "docker push shubnimkar/pet-clinic25:latest"
                    }
                }
            }
        }
        
	    
	stage("Deploy To Tomcat"){
            steps{
                sh "cp  /var/lib/jenkins/workspace/DevSecOps/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }

	    
	stage ("Dynamic Analysis - DAST with OWASP ZAP") {
			steps {
			sshagent(['SSH-Cred']){
				sh 'ssh ubuntu@13.232.127.89 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t  owasp/zap2docker-stable zap-baseline.py -t http://3.108.238.36:8081/petclinic/ -J zap_report.json || true" '
				archiveArtifacts artifacts: 'zap_report.html', allowEmptyArchive: true

            }
        }
    }
    }
}






	




    

