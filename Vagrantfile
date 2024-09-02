# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  nodes = [
#    { :hostname => "md-u", :mac => "5254009f0a05", :ram => 10240 },
    { :hostname => "ps1u", :mac => "5254009f0a01", :ram => 6144 },
    { :hostname => "ps2u", :mac => "5254009f0a02", :ram => 6144 },
    { :hostname => "ps3u", :mac => "5254009f0a03", :ram => 6144 }
  ]

  nodes.each do |node|
    config.vm.define node[:hostname] do |node_config|
      node_config.vm.box = "generic/rocky9"
      node_config.vm.network :public_network, :mac => node[:mac], bridge: "eno3"
      node_config.vm.hostname = node[:hostname]
      node_config.vm.boot_timeout = 600

node_config.vm.provision "shell", inline: <<-SHELL
  timedatectl set-timezone America/Los_Angeles
  mkdir -p /home/vagrant/.ssh
  cp /home/vagrant/.ssh/id_ed25519.pub /home/vagrant/.ssh/authorized_keys
  chmod 600 /home/vagrant/.ssh/authorized_keys
  chown vagrant:vagrant /home/vagrant/.ssh/authorized_keys

  # Install PerfSONAR
  curl -s https://downloads.perfsonar.net/install | sh -s - --auto-updates --tunings testpoint
 SHELL
  
#      node_config.vm.provision "shell" do |shell|
#        shell.inline = "timedatectl set-timezone America/Los_Angeles"
#        shell.inline =X "file", source: "/home/vagrant/.ssh/id_ed25519.pub", destination: "/home/vagrant/.ssh/authorized_keys"
#      end

      node_config.vm.provider :virtualbox do |domain|
        domain.memory = node[:ram]
        domain.cpus = 1
      end
    end
  end

#  config.vm.provision "ansible" do |ansible|
#    ansible.compatibility_mode = "2.0"
#    ansible.playbook = "Testpoint-MD-buildR9.yml"
#    ansible.groups = {
#      "testpoint" => ["ps5-1", "ps5-2", "ps5-3"],
#      "md-arch"   => ["md-5"]
#    }
#  end
end
