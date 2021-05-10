# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.define "gitlab" do |gitlab|
        gitlab.vm.hostname = "gitlab.example.com"
        gitlab.vm.box = "generic/centos7"
        gitlab.vm.network "forwarded_port", guest: 80, host:8080, host_ip: "127.0.0.1"
        gitlab.vm.network "private_network", ip: "10.10.10.12"
        gitlab.vm.synced_folder ".", "/vagrant", type: "rsync"
        gitlab.vm.provision "shell", inline: <<-SHELL
            
            #Gitlab Setup
            
            sudo yum install -y curl policycoreutils-python openssh-server perl
            sudo systemctl enable sshd
            sudo systemctl start sshd
            sudo firewall-cmd --permanent --add-service=http
            sudo firewall-cmd --permanent --add-service=https
            sudo systemctl reload firewalld

            curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
            sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee

            # Setup DNS client with vagranting-dns
            ETH0=$(sudo nmcli connection show | grep eth0 | cut -d ' ' -f 4)
            ETH1=$(sudo nmcli connection show | grep eth1 | cut -d ' ' -f 4)
            sudo nmcli connection modify $ETH1 ipv4.dns 10.10.10.2
            sudo nmcli connection modify $ETH1 ipv4.dns-search example.com
            sudo nmcli connection modify $ETH1 ipv4.dns-priority 1
            sudo nmcli connection modify $ETH0 ipv4.dns-priority 10
            sudo nmcli connection up $ETH1
            sudo nmcli connection up $ETH0
        SHELL
        gitlab.vm.provider "virtualbox" do |vb|
            vb.cpus = "1"
            vb.memory = "1024"
            vb.customize ["modifyvm", :id, "--groups", "/vagranting"] #If you have problems with folder exist error -> Comment out this line
            vb.name = "gitlab"
        end
    end
end

