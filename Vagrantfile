Vagrant.configure("2") do |config|
    # Shared VM box
    config.vm.box = "ubuntu/jammy64"
  
    # Load Balancer
    config.vm.define "lb" do |lb|
      lb.vm.hostname = "lb"
      lb.vm.network "private_network", ip: "192.168.70.100"
      lb.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      lb.vm.provision "shell", path: "lb-setup.sh"
    end
  
    # Web servers loop
    (1..3).each do |i|
      config.vm.define "web#{i}" do |web|
        web.vm.hostname = "web#{i}"
        web.vm.network "private_network", ip: "192.168.70.10#{i}"
        web.vm.provider "virtualbox" do |vb|
          vb.memory = "1024"
        end
        web.vm.synced_folder "./frontend/build", "/var/www/tour-app"
        web.vm.provision "shell", inline: <<-SHELL
          sudo apt update
          sudo apt install -y nginx nodejs npm
          cd /vagrant/frontend && npm install && npm run build
          sudo rm -rf /var/www/html
          sudo ln -s /var/www/tour-app /var/www/html
        SHELL
      end
    end
  end
  