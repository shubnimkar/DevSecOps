# Install Tomcat

	/opt sudo wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.65/bin/apache-tomcat-9.0.65.tar.gz
	
	sudo tar -xvf apache-tomcat-9.0.65.tar.gz
	
	cd /opt/apache-tomcat-9.0.65/conf
	
	sudo vi tomcat-users.xml
	
	---add-below-line at the end (2nd-last line)----
	<user username="admin" password="admin1234" roles="admin-gui, manager-gui"/>
	
	sudo ln -s /opt/apache-tomcat-9.0.65/bin/startup.sh /usr/bin/startTomcat
	sudo ln -s /opt/apache-tomcat-9.0.65/bin/shutdown.sh /usr/bin/stopTomcat
	
	sudo vi /opt/apache-tomcat-9.0.65/webapps/manager/META-INF/context.xml
	comment:
	  <!-- Valve className="org.apache.catalina.valves.RemoteAddrValve"
	  allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
	  
	sudo vi /opt/apache-tomcat-9.0.65/webapps/host-manager/META-INF/context.xml
	comment:
	  <!-- Valve className="org.apache.catalina.valves.RemoteAddrValve"
	  allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
	
	sudo stopTomcat
	
	sudo startTomcat
	
	sudo  cp /home/ubuntu/myagent/_work/5/s/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/


# JDK & Maven Installation 

	sudo apt-get update -y
	sudo apt install openjdk-11-jre -y
	sudo apt-get install maven -y



# Docker Installation

	sudo apt-get update
 
	sudo apt-get install ca-certificates curl gnupg
	
	sudo install -m 0755 -d /etc/apt/keyrings
 
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
 
	sudo chmod a+r /etc/apt/keyrings/docker.gpg
	
	echo \
	  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
	  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
	  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	  
	  sudo apt-get update
	  
	  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
	  
	service docker restart
 
	sudo usermod -aG docker $USER
 
	newgrp docker
 
	sudo chmod 666 /var/run/docker.sock
 
	sudo systemctl restart docker


# INSTALL JENKINS 

	sudo apt install openjdk-11-jre
	
	curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
	  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
	  
	echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
	  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
	  /etc/apt/sources.list.d/jenkins.list > /dev/null
	
	sudo apt-get update -y 
	
	sudo apt-get install jenkins -y
	
	sudo systemctl enable jenkins
	
	sudo systemctl start jenkins
	
	sudo systemctl status jenkins

# SONARQUBE INSTALLATION STEPS

	sudo apt update -y
	
	sudo apt-get install \
	    ca-certificates \
	    curl \
	    gnupg \
	    lsb-release
	    
	sudo mkdir -m 0755 -p /etc/apt/keyrings
	
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
	
	echo \
	  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
	  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	
	sudo apt update -y
	
	sudo apt-get install docker-ce docker-ce-cli docker-compose -y
	
	sudo vi /etc/sysctl.conf
	
	---In the Config file, press the “I” key to insert content and add the below lines at the end of the file and save the changes.---
	
	vm.max_map_count=262144
	fs.file-max=65536
	
	sudo sysctl -p
	
	sudo groupadd docker
	sudo usermod -aG docker ubuntu
	
	---Create a docker-compose file and start the Sonar server---
	
	version: "3"
	
	services:
	  sonarqube:
	    image: sonarqube:community
	    depends_on:
	      - db
	    environment:
	      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
	      SONAR_JDBC_USERNAME: sonar
	      SONAR_JDBC_PASSWORD: sonar
	    volumes:
	      - sonarqube_data:/opt/sonarqube/data
	      - sonarqube_extensions:/opt/sonarqube/extensions
	      - sonarqube_logs:/opt/sonarqube/logs
	    ports:
	      - "9000:9000"
	  db:
	    image: postgres:12
	    environment:
	      POSTGRES_USER: sonar
	      POSTGRES_PASSWORD: sonar
	    volumes:
	      - postgresql:/var/lib/postgresql
	      - postgresql_data:/var/lib/postgresql/data
	
	volumes:
	  sonarqube_data:
	  sonarqube_extensions:
	  sonarqube_logs:
	  postgresql:
	  postgresql_data:
	
	---For this, I will create a docker-compose.yml using the below command---
	
	nano docker-compose.yml
	
	---we can access our server---
	
	docker-compose up -d
	
	---In order to tear down the complete infra, we can run the below command which will remove all the containers and our server will no longer be accessible.---
	
	docker-compose down


# TRIVY INSTALLATION STEPS

	sudo apt-get install wget apt-transport-https gnupg lsb-release
	
	wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
	
	echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
	
	sudo apt-get update
	
	sudo apt-get install trivy -y
