# -*- mode: ruby -*-
# vi: set ft=ruby :

# TODO Add necessary minimum provisioning
$script = <<EOF
mkdir -pv /data/testarteria1
mkdir /data/testarteria1/mon1
mkdir /data/testarteria1/mon2
chown -R vagrant:vagrant /data
chmod -R g+w /data
mkdir -pv /data/scratch

echo "Installing basic global python requirements"
apt-get -y install python-pip
pip install virtualenv

echo "Installing the runfolder package"
cd /arteria/arteria-lib/runfolder/scripts/
./install vagrant vagrant

EOF

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  # Ensure that the VMs can reach each other
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false

  # Configure the stackstorm master node
  config.vm.define "stackstorm-master" do |stackstorm|

    stackstorm.vm.hostname = "stackstorm-master"
    stackstorm.vm.network :private_network, ip: '192.168.42.42'

    stackstorm.vm.synced_folder "../arteria-packs/", "/opt/stackstorm/packs/arteria-packs"
    stackstorm.vm.synced_folder "../arteria-packs/", "/arteria/arteria-packs"
    stackstorm.vm.synced_folder "../arteria-lib/", "/arteria/arteria-lib"

    # Forwarding these ports is required for the webui to work
    stackstorm.vm.network :forwarded_port, host: 8080, guest: 8080
    stackstorm.vm.network :forwarded_port, host: 9100, guest: 9100
    stackstorm.vm.network :forwarded_port, host: 9101, guest: 9101

    stackstorm.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end

    stackstorm.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible-st2/playbooks/arteriaexpress.yaml"
      ansible.sudo = true
    end

  end

  # Configure the biotank stand-in
  config.vm.define "testarteria1" do |testtank|
    testtank.vm.hostname = "testarteria1"
    testtank.vm.network :private_network, ip: '192.168.42.43'

    testtank.vm.synced_folder "../arteria-packs/", "/arteria/arteria-packs"
    testtank.vm.synced_folder "../arteria-lib/", "/arteria/arteria-lib"

    testtank.vm.provision "shell", inline: $script
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

  config.vm.provision "shell", inline: "ssh-keygen -y -f #{vagrant_ssh}/key > #{vagrant_ssh}/key.pub"
  config.vm.provision "shell", inline: "cat #{vagrant_ssh}/key.pub >> #{vagrant_ssh}/authorized_keys"

end
