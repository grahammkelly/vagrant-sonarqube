# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANT_API_VER = "2"
VM_NAME = "sonarqube-test-server"
Vagrant.configure(VAGRANT_API_VER) do |config|
  config.vm.box = "ubuntu/wily64"
  config.vm.hostname = VM_NAME

  # config.vm.network "private_network", ip: "192.168.99.101"
  config.vm.network :forwarded_port, guest: 9000, host: 9000

  config.vm.provider "virtualbox" do |vb|
  	vb.name = VM_NAME
    vb.memory = "1024"
  end

  if (File.exist?("#{Dir.pwd}/data"))
  	config.vm.synced_folder "#{Dir.pwd}/data", "/vagrant_data"
  end

  config.vm.provision :shell do |sh|
  	ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip

    sh.inline = <<-SHELL
		#Give SSH access to the box to the user provisioning
		echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
		echo £{ssh_pub_key} >> /root/.ssh/authorized_keys

		#Install and configure Java 8
		set -e -x
		apt-get update
		apt-get -y install openjdk-8-jre-headless

		# most of this is courtesy of
		# https://github.com/SonarSource/docker-sonarqube/blob/master/5.2/Dockerfile

		# Install and configure SonarQube
		cat >/run/systemd/system/sonarqube.env<<ENV
		SONARQUBE_HOME=/opt/sonarqube
		SONARQUBE_JDBC_USERNAME=sonar
		SONARQUBE_JDBC_PASSWORD=sonar
		SONARQUBE_JDBC_URL='jdbc:h2:tcp://localhost:9092/sonar'
		SONAR_VERSION=5.2
		ENV
		source /run/systemd/system/sonarqube.env

		# pub   2048R/D26468DE 2015-05-25
		#       Key fingerprint = F118 2E81 C792 9289 21DB  CAB4 CFCA 4A29 D264 68DE
		# uid                  sonarsource_deployer (Sonarsource Deployer) <infra@sonarsource.com>
		# sub   2048R/06855C1D 2015-05-25
		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys F1182E81C792928921DBCAB4CFCA4A29D26468DE \
		    && cd /opt \
		    && curl -o sonarqube.zip -sfSL https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-$SONAR_VERSION.zip \
		    && curl -o sonarqube.zip.asc -fSL https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-$SONAR_VERSION.zip.asc \
		    && gpg --verify sonarqube.zip.asc \
		    && unzip sonarqube.zip \
		    && mv sonarqube-$SONAR_VERSION sonarqube \
		    && chown -R vagrant sonarqube \
		    && rm sonarqube.zip* \
		    && rm -rf $SONARQUBE_HOME/bin/*

		cd $SONARQUBE_HOME
		cat >/etc/systemd/system/sonarqube.service<<SVC
		[Unit]
		Wants=network-online.target
		[Install]
		WantedBy=multi-user.target
		[Service]
		User=vagrant
		WorkingDirectory=/opt/sonarqube
		ExecStart=/usr/bin/java -jar lib/sonar-application-$SONAR_VERSION.jar \\
		  -Dsonar.log.console=true \\
		  -Dsonar.jdbc.username="$SONARQUBE_JDBC_USERNAME" \\
		  -Dsonar.jdbc.password="$SONARQUBE_JDBC_PASSWORD" \\
		  -Dsonar.jdbc.url="$SONARQUBE_JDBC_URL" \\
		  -Dsonar.web.javaAdditionalOpts="-Djava.security.egd=file:/dev/./urandom"
		SVC
		systemctl enable sonarqube.service
		systemctl start sonarqube.service
		SHELL
  end
end

