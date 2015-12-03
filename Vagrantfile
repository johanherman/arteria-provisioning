# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Ensure that the VMs can reach each other
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false

  # Configure the stackstorm master node
  config.vm.define "stackstorm-master" do |stackstorm|

    stackstorm.vm.box = "ubuntu/trusty64"
    stackstorm.vm.box_version = "20150909.1.0"

    stackstorm.vm.hostname = "stackstorm-master"
    stackstorm.vm.network :private_network, ip: '192.168.42.42'

    stackstorm.vm.synced_folder "../arteria-packs/", "/opt/stackstorm/packs/arteria-packs"
    stackstorm.vm.synced_folder "../arteria-packs/", "/arteria/arteria-packs"
    stackstorm.vm.synced_folder "../arteria-core/", "/arteria/arteria-core"
    stackstorm.vm.synced_folder "../arteria-bcl2fastq/", "/arteria/arteria-bcl2fastq"
    stackstorm.vm.synced_folder "../arteria-siswrap/", "/arteria/arteria-siswrap"
    stackstorm.vm.synced_folder "../arteria-runfolder/", "/arteria/arteria-runfolder"
    stackstorm.vm.synced_folder "../arteria-provisioning/", "/arteria/arteria-provisioning"

    stackstorm.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.customize ["modifyvm", :id, "--ioapic", "on"]
    end

    stackstorm.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible-st2/playbooks/arteriaexpress.yaml"
      ansible.inventory_path = "ansible-st2/inventories/test_inventory"
    end

  end

  # Configure the biotank stand-in
  config.vm.define "testarteria1" do |testtank|

    testtank.vm.box = "puppetlabs/centos-6.6-64-puppet"

    testtank.vm.provider "virtualbox" do |v|
      v.memory = 8000
      v.cpus = 4
      v.customize ["modifyvm", :id, "--ioapic", "on"]
    end

    testtank.vm.hostname = "testarteria1"
    testtank.vm.network :private_network, ip: '192.168.42.43'

    testtank.vm.synced_folder "../arteria-packs/", "/arteria/arteria-packs"
    testtank.vm.synced_folder "../arteria-core/", "/arteria/arteria-core"
    testtank.vm.synced_folder "../arteria-bcl2fastq/", "/arteria/arteria-bcl2fastq"
    testtank.vm.synced_folder "../arteria-siswrap/", "/arteria/arteria-siswrap"
    testtank.vm.synced_folder "../arteria-runfolder/", "/arteria/arteria-runfolder"
    testtank.vm.synced_folder "../arteria-provisioning/", "/arteria/arteria-provisioning"
    #testtank.vm.synced_folder "/data/arteria_test_data/", "/data/testarteria1/runfolders"

    # By default this box has iptables enabled on all ports
    # since the Ansible provisioning does not handle firewall configuration
    # at the moment we need to ensure that all services can be accessed
    # anyway. We do this by dropping the reject all rule here:
    config.vm.provision "shell", inline: "sudo iptables -D INPUT 5; echo 0;"

    testtank.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible-st2/playbooks/arteriaexpress.yaml"
      ansible.inventory_path = "ansible-st2/inventories/test_inventory"
    end

  end

  # Configure the uppmax stand-in
  config.vm.define "testuppmax" do |testuppmax|

    testuppmax.vm.box = "puppetlabs/centos-6.6-64-puppet"
    testuppmax.vm.hostname = "testuppmax"
    testuppmax.vm.network :private_network, ip: '192.168.42.44'

    testuppmax.vm.provision "shell", inline: "mkdir -p /tmp/runfolders"
    testuppmax.vm.provision "shell", inline: "chown -R vagrant:vagrant /tmp/runfolders"
    testuppmax.vm.provision "shell", inline: "mkdir -p /tmp/summaries"
    testuppmax.vm.provision "shell", inline: "chown -R vagrant:vagrant /tmp/summaries"

    testuppmax.vm.provision "shell", inline: 'echo export SNIC_RESOURCE=milou >> /home/vagrant/.bash_profile'
    testuppmax.vm.provision "shell", inline: "printf '#!/bin/bash\necho $@\n' > /usr/local/bin/sbatch"
    testuppmax.vm.provision "shell", inline: "chmod +x /usr/local/bin/sbatch"

    testuppmax.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible-st2/playbooks/arteriaexpress.yaml"
      ansible.inventory_path = "ansible-st2/inventories/test_inventory"
    end
  end

  # Deploy ssh keys to all host to ensure they have the
  # password less ssh-access to each other.

  vagrant_ssh = "/home/vagrant/.ssh"

  config.vm.provision "file",
    source: "private_key",
    destination: vagrant_ssh + "/key"

  config.vm.provision "file",
    source: "ssh_config",
    destination: vagrant_ssh + "/config"

  config.vm.provision "shell", inline: "chmod og-rwx #{vagrant_ssh}/config"
  config.vm.provision "shell", inline: "ssh-keygen -y -f #{vagrant_ssh}/key > #{vagrant_ssh}/key.pub"
  config.vm.provision "shell", inline: "cat #{vagrant_ssh}/key.pub >> #{vagrant_ssh}/authorized_keys"

end
