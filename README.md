# 🚀 TourPlaces Deployment using Vagrant Cluster

👩‍💻 Full Name: **Renad Alreahili**

---

## 📦 Project Structure

```
.
├── frontend/           # Cloned React app
├── Vagrantfile         # VM setup configuration
├── lb-setup.sh         # NGINX Load Balancer script
└── images/             # Screenshots (vagrant status, curl outputs)
```

---

## 🌐 Application Background

- React travel app: [TourPlaces](https://github.com/cw-barry/Tour-Places-App.git)

---

## 🛠️ Technologies Used

- Vagrant + VirtualBox 🧱
- Ubuntu 22.04 LTS 🐧
- Node.js & npm 🟢
- NGINX Web Server 🌐
- Shell Scripting 🔧

---

## 🧱 Part 1: Cluster Setup – Vagrantfile

### 🔗 Clone the app:
```bash
git clone https://github.com/cw-barry/Tour-Places-App.git frontend
```

### 🛠️ Vagrantfile content:
```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "lb" do |lb|
    lb.vm.hostname = "lb"
    lb.vm.network "private_network", ip: "192.168.70.100"
    lb.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
    end
    lb.vm.provision "shell", path: "lb-setup.sh"
  end

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
```

📸 Screenshot:  
![vagrant status](images/vagrant-status.png)

---

## 🌐 Part 2: Load Balancer Configuration

### 🧾 lb-setup.sh
```bash
#!/bin/bash

sudo apt update
sudo apt install -y nginx

cat <<EOF | sudo tee /etc/nginx/conf.d/tour-places.conf
upstream react_servers {
    server 192.168.70.101;
    server 192.168.70.102;
    server 192.168.70.103;
}

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://react_servers;
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
}
EOF

sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

📸 Screenshot:  
![curl -I](images/curl-lb.png)

---

## 💥 Part 3: Failure Testing

```bash
vagrant halt web2
for i in {1..10}; do curl http://192.168.70.100; done
```

📸 Screenshot:  
![curl after failure](images/curl-failover.png)

---

## 📈 Part 4: Scaling (Add Web4)

```ruby
config.vm.define "web4" do |web|
  web.vm.hostname = "web4"
  web.vm.network "private_network", ip: "192.168.70.104"
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
```

```bash
sed -i '/upstream react_servers {/a     server 192.168.70.104;' /etc/nginx/conf.d/tour-places.conf
sudo nginx -t && sudo systemctl reload nginx
```

📸 Screenshot:  
![curl after scaling](images/curl-scaled.png)

---



