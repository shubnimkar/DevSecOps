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
              sh 'docker run gesellix/trufflehog --json https://github.com/shubnimkar/CI_CD_Devsecops.git > trufflehog'
              sh 'cat trufflehog'
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
          

	    stage ('Dynamic analysis') {
            steps {
           sshagent(['SSH-Cred']) {

		   sh 'ssh ubuntu@13.232.127.89 "sudo /opt/zap/zap.sh -cmd -quickurl http://3.108.238.36:8081/petclinic -quickout ~/out.xml" '
		   //sh 'ssh ubuntu@13.232.127.89 "sudo /opt/zap/zap.sh -exportreport zap-report.json -reportformat JSON"'
		   //sh "scp ubuntu@13.232.127.89:/opt/zap-report.json ."
                  // sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.127.89 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://3.108.238.36:8081/petclinic -x zap_report || true" '
		   //sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.127.89 "sudo ./zap_report.sh"'  
		   //sh 'sudo ssh root@13.232.127.89 "sudo docker run -v :/zap/wrk/:rw -t ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py -t https://3.108.238.36:8081/petclinic -g gen.conf -r testreport.html " '
				 
	//	    sh 'sudo ssh root@13.232.127.89 "sudo docker run --rm -u root -v /opt/zap:/zap/wrk:rw -t owasp/zap2docker-stable zap-baseline.py -t http://3.108.238.36:8081/petclinic -x zap_report || true" '
        
              }      
           }
	//post {
        //always {
            // Archive the ZAP report as a build artifact
          //  archiveArtifacts artifacts: '**/zap-report.json', allowEmptyArchive: true
        //}
//}
}
}
}
    

