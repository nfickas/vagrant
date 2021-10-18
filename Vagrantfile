# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_MEM = 4096
VM_CPU = 2
VM_NAME = "ha"
servers = Array[
  "ha-etcd-1",
  "ha-pg-1",
  "ha-pg-2"
]
ips = Array[
  "192.168.4.26",
  "192.168.4.27",
  "192.168.4.28"
]

servers.each_with_index { |server, index|
  Vagrant.configure("2") do |config|
    ssh_pub_key = File.readlines("./vagrant_centos/keys/id_rsa.pub").first.strip
    config.vm.box = "centos/7"

    config.vm.provision "shell", path: "setup_ssh.sh"
  
    config.vm.define server do |srv|
      srv.vm.hostname = server
      srv.vm.network "private_network", ip: ips[index]
      srv.trigger.after :up do |trigger|
        trigger.name = "Setup Known Host"
        trigger.info = "ssh's into vm and accepts the vm's fingerprint"
        trigger.run = {inline: "./root_login.sh #{ips[index]}"}
        trigger.exit_codes = [255]
      end
      
  
      srv.vm.provider :libvirt do |kvm|
        kvm.name = server
        kvm.memory = VM_MEM
        kvm.cpus = VM_CPU
        kvm.gui = false
        kvm.linked_clone=true
      end
  
      srv.vm.provision "shell" do |s|
        s.inline = <<-SHELL
          sudo echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          sudo echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      end
    end
  end
}

# Setup SSH Keys
system("
    if [[ #{ARGV[0]} = 'up' ]]
    then
        echo 'Moving ssh keys to known_hosts file...'
        sudo cp ./vagrant_centos/keys/id_rsa.pub ~/.ssh/vagrant.pub
        sudo cp ./vagrant_centos/keys/id_rsa ~/.ssh/vagrant
    fi
")
