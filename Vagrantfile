
$TOMCAT_COUNT = 2

Vagrant.configure("2") do |config|
	
	config.vm.box = "bertvv/centos72"
	config.vm.provider "virtualbox" do |vb|
		vb.gui = true
	end
  
	config.vm.define "apache1" do |apache| 
		apache.vm.hostname = "apache1"
		apache.vm.network "forwarded_port", guest: 80, host: 8080
		apache.vm.network "public_network", ip: "192.168.0.10"
		apache.vm.provision "shell", inline: <<-SHELL
			
			yum install httpd -y
			yum install mc -y
			yum install git -y
			
			firewall-cmd --zone=public --add-port=80/tcp --permanent
			firewall-cmd --reload
			systemctl stop firewald			

			rm -rf /git
			git clone https://github.com/AliakseiUstsinau/Moodule-2 /git
			cd /git
			git checkout task3
			rm /etc/httpd/modules/mod_jk.so
			cp /git/mod_jk.so /etc/httpd/modules/
			
			# httpd.conf
			sed -i '/mod_jk.so/d' /etc/httpd/conf/httpd.conf
			sed -i '/workers.properties/d' /etc/httpd/conf/httpd.conf
			sed -i '/JkShmFile/d' /etc/httpd/conf/httpd.conf
			sed -i '/JkLogFile/d' /etc/httpd/conf/httpd.conf
			sed -i '/JkLogLevel/d' /etc/httpd/conf/httpd.conf
			sed -i '/JkMount/d' /etc/httpd/conf/httpd.conf			

			echo 'LoadModule jk_module modules/mod_jk.so' >> /etc/httpd/conf/httpd.conf
			echo 'JkWorkersFile conf/workers.properties' >> /etc/httpd/conf/httpd.conf
			echo 'JkShmFile /tmp/shm' >> /etc/httpd/conf/httpd.conf
			echo 'JkLogFile logs/mod_jk.log' >> /etc/httpd/conf/httpd.conf
			echo 'JkLogLevel info' >> /etc/httpd/conf/httpd.conf
			echo 'JkMount /1 lb' >> /etc/httpd/conf/httpd.conf
			
			# workers.properties

			rm /etc/httpd/conf/workers.properties
			echo 'worker.list=lb' >> /etc/httpd/conf/workers.properties
			echo 'worker.lb.type=lb' >> /etc/httpd/conf/workers.properties
			echo 'worker.lb.balance_workers=tomcat1' >> /etc/httpd/conf/workers.properties
		SHELL
		
		(2..$TOMCAT_COUNT).each do |i|
			apache.vm.provision "shell", inline: <<-SHELL
				sed -i '/balance/s/$/,tomcat#{i}/' /etc/httpd/conf/workers.properties
			SHELL
		end	
		(1..$TOMCAT_COUNT).each do |i|	
			apache.vm.provision "shell", inline: <<-SHELL
					echo 'worker.tomcat#{i}.host=192.168.0.#{10+i}' >> /etc/httpd/conf/workers.properties
					echo 'worker.tomcat#{i}.port=8009' >> /etc/httpd/conf/workers.properties
					echo 'worker.tomcat#{i}.type=ajp13' >> /etc/httpd/conf/workers.properties
			SHELL
		end
		(1..$TOMCAT_COUNT).each do |i|
			apache.vm.provision "shell", inline: <<-SHELL
				echo -e "192.168.0.#{10+i}	tomcat#{i}" >> /etc/hosts
			SHELL
		end	

		apache.vm.provision "shell", inline: <<-SHELL
			/etc/init.d/network restart
			systemctl enable httpd
			systemctl start httpd
		SHELL
	end



# TOMCATs
	
	(1..$TOMCAT_COUNT).each do |i|
						
		config.vm.define "tomcat#{i}" do |tomcat|
			tomcat.vm.hostname = "tomcat#{i}"
			tomcat.vm.network "public_network", ip: "192.168.0.#{10+i}"
			tomcat.vm.provision "shell", inline: <<-SHELL
				yum install mc -y
				yum install java-1.8.8-openjdk -y
				yum install tomcat tomcat-webapps tomcat-admin-webapps -y

				sed -i '/apache1/d' /etc/hosts
				echo -e '192.168.0.10	apache1' >> /etc/hosts

				sed -i 's/.*Engine name="Catalina"  defaultHost="localhost"*./<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat#{i}">/' /usr/share/tomcat/conf/server.xml
				
				mkdir /usr/share/tomcat/webapps/1
				rm /usr/share/tomcat/webapps/1/index.html
				echo '<h1>Tomcat#{i}</h1>' >> /usr/share/tomcat/webapps/1/index.html			

				/etc/init.d/network restart
				systemctl enable tomcat
				systemctl start tomcat
			SHELL
		end
	end
end
