# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos/7"
  config.vm.hostname = "itesla"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "itesla"
    vb.memory = 1024
    vb.cpus = 2
  end

  config.vm.synced_folder ".", "/vagrant"
  
#  ENV['PYTHONIOENCODING'] = "utf-8"

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
mkdir -p /home/vagrant/roles
cp -r /vagrant/roles /home/vagrant

mkdir -p /home/vagrant/group_vars
cp /vagrant/group_vars_ipst_hosts.example /home/vagrant/group_vars/all

cp /vagrant/ipst.yml /home/vagrant
SHELL

  config.vm.provision :ansible_local do |ansible|
    ansible.playbook = "ipst.yml"
    ansible.provisioning_path = "/home/vagrant"
    ansible.extra_vars = {
        user_hades_src: '/vagrant'
    }
  end

  config.vm.provision :ansible_local do |ansible|
    ansible.playbook = "ipst.yml"
    ansible.provisioning_path = "/home/vagrant"
    ansible.extra_vars = {
        installNordic44: true
    }
    ansible.tags = ["Nordic44"]
  end

end