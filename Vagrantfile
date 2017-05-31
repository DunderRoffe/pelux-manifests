# -*- mode: ruby -*-
# vi: set ft=ruby :

# Path to extra disk
disk_name = 'yocto_disk.vdi'
disk_size_gb = 200

# System resources
ram = 16 #GB
cpus = 6

Vagrant.configure(2) do |config|

    config.vm.box = "debian/jessie64"

    config.vm.provider "virtualbox" do |vb|
        vb.memory = ram * 1024
        vb.cpus = cpus
    end

    # Attach an extra disk to /home/vagrant
    eval (File.read 'vagrant-cookbook/host-system-config/attach-extra-disk.vagrantfile')
    config.vm.provision "shell", args: ["/dev/sdb"], path: "vagrant-cookbook/host-system-config/create-disk.sh"

    # On some hosts, the network stack needs to be kicked alive
    config.vm.provision "shell", inline: <<-SHELL
        ping google.com &> /dev/null &
    SHELL

    # Install dependencies for BitBake
    config.vm.provision "shell", path: "vagrant-cookbook/deps/yocto.sh"

    # Not all networks can handle IPv6, so we disable it for now
    config.vm.provision "shell", path: "vagrant-cookbook/system-config/disable-ipv6.sh"

    # Configure username and password in git
    config.vm.provision "shell", privileged: false, path: "vagrant-cookbook/system-config/vagrant-ssh-user.sh"

    # Configure username and password in git
    config.vm.provision "shell", privileged: false, path: "vagrant-cookbook/yocto/initialize-repo-tool.sh"
end

