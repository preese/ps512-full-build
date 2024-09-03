# -*- mode: ruby -*-
# vi: set ft=ruby :

nodes = [
    { :hostname => "md-u", :mac => "5254009f0a05", :ram => 10240 },
    { :hostname => "ps1u", :mac => "5254009f0a01", :ram => 6144 },
    { :hostname => "ps2u", :mac => "5254009f0a02", :ram => 6144 },
    { :hostname => "ps3u", :mac => "5254009f0a03", :ram => 6144 }
]

Vagrant.configure("2") do |config|
  nodes.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.vm.box = "generic/rocky9"
      node_config.vm.network :public_network, :mac => node[:mac], bridge: "enp0s20f0u2"
      node_config.vm.hostname = node[:hostname]
      node_config.vm.boot_timeout = 600

      # Provision the file
      node_config.vm.provision "file", source: "/home/vagrant/transfer", destination: "/home/vagrant/transfer"

      # Define multiple commands based on hostname
      curl_command = if node[:hostname] == "md-u"
        <<-SHELL
          curl -s https://downloads.perfsonar.net/install | sh -s - --auto-updates archive
          sed -i '/# Require ip 10.1.1.0\/24/i Require ip 192.168.0.0/23' /etc/httpd/conf.d/apached-logstash.conf
          systemctl restart httpd
          dnf -y install perfsonar-grafana perfsonar-grafana-toolkit perfsonar-psconfig-hostmetrics perfsonar-psconfig-publisher
          firewall-cmd --perm --add-service=https
          firewall-cmd --reload
          psconfig publish /home/vagrant/transfer/3by3.json
          psconfig remote add "https://md-u.ufixu.com/psconfig/3by3.json"
        SHELL
      else
        <<-SHELL
          curl -s https://downloads.perfsonar.net/install | sh -s - --auto-updates --tunings default
          dnf install -y perfsonar-toolkit-security
          /usr/lib/perfsonar/scripts/configure_firewall install
          psconfig remote add "https://md-u.ufixu.com/psconfig/3by3.json"
        SHELL
      end

      # Provisioning shell script with multiple commands
      node_config.vm.provision "shell", inline: <<-SHELL
        echo "Setting timezone and preparing SSH..." >> /var/log/vagrant_provision.log
        timedatectl set-timezone America/Los_Angeles
        mkdir -p /home/vagrant/.ssh
        cp /home/vagrant/transfer/id_ed25519.pub /home/vagrant/.ssh/authorized_keys
        chmod 600 /home/vagrant/.ssh/authorized_keys
        chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys
        echo "Executing curl command..." >> /var/log/vagrant_provision.log
        #{curl_command}
      SHELL

      node_config.vm.provider :virtualbox do |domain|
        domain.memory = node[:ram]
        domain.cpus = 1
      end
    end
  end
end

