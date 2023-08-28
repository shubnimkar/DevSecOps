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
        /*stage ("Dynamic Analysis - DAST with OWASP ZAP") { 
            steps {
                sh 'mkdir -p /opt/zap'
                sh "docker run --rm -u root -v /opt/zap:/zap/wrk:rw -t owasp/zap2docker-stable zap-baseline.py -t http://3.108.238.36:8081/petclinic -x zap_report || true"
                /*sh 'docker pull owasp/zap2docker-stable'

                sh "docker run -dt --name owasp owasp/zap2docker-stable /bin/bash" 
                sh """
                docker exec owasp \
                mkdir /zap/wrk
                """
                sh "docker exec owasp zap-baseline.py -t http://3.108.238.36:8081/petclinic -x zap_report || true"
                //sh "docker cp owasp:/zap/wrk/report.json /var/lib/jenkins/zap/wrk/report.json" 
                sh "docker stop owasp"
                sh "docker rmi -f owasp"
                */
                
        
           /* stage('Run OWASP ZAP Scan') {
            steps {
                
		   /* sh 'docker stop owasp'
		    sh 'docker rm owasp'
                   /* // Run OWASP ZAP scan using Docker
                    sh 'docker run -v /home/ubuntu:/zap/wrk --name owasp owasp/zap2docker-stable zzap-full-scan.py -t http://3.108.238.36:8081/petclinic -J report.json || true'
                    archiveArtifacts artifacts: 'report.json', allowEmptyArchive: true */

		   
    //     	    sh  " docker run --rm -v /home/ubuntu:/zap/wrk:rw -t owasp/zap2docker-stable zap-baseline.py -t http://3.108.238.36:8081/petclinic -J zap_report.json" 
  //      	    archiveArtifacts artifacts: 'zap_report.json', allowEmptyArchive: true 
		    
//	    }
//	    }        

	    stage ('Dynamic analysis') {
            steps {
           sshagent(['SSH-Cred']) {

		   sh 'ssh ubuntu@13.232.127.89 "sudo /opt/zap/zap.sh -cmd -quickurl http://3.108.238.36:8081/petclinic " '
		   sh 'ssh ubuntu@13.232.127.89 "sudo /opt/zap/zap.sh -exportreport zap-report.json -reportformat JSON"'
		 // sh "scp ubuntu@13.232.127.89:/opt/zap-report.json ."
            	   
		   //archiveArtifacts artifacts: 'zap_report.json', allowEmptyArchive: true 
               // sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.127.89 "sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t owasp/zap2docker-stable zap-full-scan.py -t http://3.108.238.36:8081/petclinic -x zap_report || true" '
		//sh 'ssh -o  StrictHostKeyChecking=no ubuntu@13.232.127.89 "sudo ./zap_report.sh"'
              }      
           }      
}
}
}
    
