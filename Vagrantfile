
Vagrant.configure("2") do |config|
  
	config.vm.box = "bertvv/centos72"
	config.vm.provider "virtualbox" do |vb|
		vb.gui = true
	end
  
	config.vm.define "server1" do |server1|
		server1.vm.hostname = "server1"
		server1.vm.network "public_network", ip: "192.168.0.10"
		server1.vm.provision "shell", inline: <<-SHELL
			sudo yum install git -y
			sudo yum install mc -y
			sudo sed -i '/server2/d' /etc/hosts
			sudo echo -e '192.168.0.11	server2' >> /etc/hosts
			sudo /etc/init.d/network restart

			sudo rm -rf /git
			sudo git clone https://github.com/AliakseiUstsinau/Moodule-2 /git
			cd /git
			sudo git checkout task2
			cat test.txt
			
		SHELL
	end
	
	config.vm.define "server2" do |server2|
		server2.vm.hostname = "server2"
		server2.vm.network "public_network", ip: "192.168.0.11"
		server2.vm.provision "shell", inline: <<-SHELL
			sudo yum install mc -y
			sudo sed -i '/server1/d' /etc/hosts
			sudo echo -e '192.168.0.10	server1' >> /etc/hosts
			sudo /etc/init.d/network restart
		SHELL
	end

end
