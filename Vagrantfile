# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. Please don't change it 
# unless you know what you're doing.

Vagrant.configure(2) do |config|

  # nuts vm
  config.vm.define "nuts" do |nuts|
    nuts.vm.box = "ubuntu/xenial64"
    nuts.vm.host_name = "nuts"
    nuts.vm.network "private_network", ip: "192.168.100.99"

    nuts.vm.provision "shell",
      inline: "sudo apt-get update && sudo apt-get install python-pip -y"
    nuts.vm.provision "shell",
      inline: "git clone https://github.com/HSRNetwork/Nuts.git"
    nuts.vm.provision "shell",
      inline: "cd Nuts; sudo python setup.py install"

  end

  # salt master vm
  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/xenial64"
    master.vm.host_name = "saltmaster"
    master.vm.network "private_network", ip: "192.168.100.100"

    master.vm.synced_folder "saltstack/salt/", "/srv/salt"
    master.vm.synced_folder "saltstack/pillar/", "/srv/pillar"
    master.vm.synced_folder "saltstack/master.d/", "/etc/salt/master.d"

    # Copy custom module into salt
    master.vm.provision "shell",
      inline: "mkdir -p /srv/salt/_module"
    master.vm.provision "shell",
      inline: "wget -O /srv/salt/_module/nuts.py https://raw.githubusercontent.com/HSRNetwork/Nuts/develop/_modules/nuts.py"

    master.vm.provision :salt do |salt|
      salt.verbose = true
      salt.colorize = true
      salt.install_type = "git"
      # salt.install_args = "develop"
      salt.install_master = true
      # salt.master_config = "saltstack/etc/master"
      salt.bootstrap_options = "-A 192.168.100.100"

      salt.master_key = "saltstack/pki/saltmaster.pem"
      salt.master_pub = "saltstack/pki/saltmaster.pub"
      salt.minion_key = "saltstack/pki/saltmaster.pem"
      salt.minion_pub = "saltstack/pki/saltmaster.pub"

      salt.seed_master = {
                          "saltmaster" => "saltstack/keys/saltmaster.pub",
                          "minion1" => "saltstack/keys/minion1.pub",
                          "minion2" => "saltstack/keys/minion2.pub"
                         } 
    end
  end


  # salt minions
  (1..2).each do |i|
    config.vm.define "minion#{i}" do |minion|
      minion.vm.box = "ubuntu/xenial64"
      minion.vm.host_name = "minion#{i}"
      minion.vm.network "private_network", ip: "192.168.100.10#{i}"

      minion.vm.provision :salt do |salt|
        salt.verbose = true
        salt.colorize = true
        salt.install_type = "git"
        # salt.install_args = "develop"
        salt.bootstrap_options = "-A 192.168.100.100"

        salt.minion_key = "saltstack/pki/minion#{i}.pem"
        salt.minion_pub = "saltstack/pki/minion#{i}.pub"
    end

    end
  end
end
