# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_MEM = 4096
VM_CPU = 2
VM_NAME = "ha"
servers = Hash[
  "ha-etcd-1" => "192.168.4.26",
  "ha-pg-1" => "192.168.4.27",
  "ha-pg-2" => "192.168.4.28"
]

# Setup SSH Keys
system("
    if [[ #{ARGV[0]} = 'up' ]]
    then
        echo 'Moving ssh keys to known_hosts file...'
        sudo cp ./vagrant_centos/keys/id_rsa.pub ~/.ssh/vagrant.pub
        sudo cp ./vagrant_centos/keys/id_rsa ~/.ssh/vagrant
    elif [[ #{ARGV[0]} = 'destroy' ]]
    then
        echo 'Removing ssh key from known_hosts file ...'
        for i in #{servers.values}; do
          ssh-keygen -R $(echo $i | sed 's/[][,]//g')
        done
    fi
")

servers.each { |server, ip|
  Vagrant.configure("2") do |config|
    ssh_pub_key = File.readlines("./vagrant_centos/keys/id_rsa.pub").first.strip
    config.vm.box = "centos/7"

    config.vm.provision "shell", path: "setup_ssh.sh"
  
    config.vm.define server do |srv|
      srv.vm.hostname = server
      srv.vm.network "private_network", ip: ip
      srv.trigger.after :up do |trigger|
        trigger.name = "Setup Known Host"
        trigger.info = "ssh's into vm and accepts the vm's fingerprint"
        trigger.run = {inline: "./root_login.sh #{ip}"}
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
