# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrant multi machine configuration

require 'yaml'
config_yml = YAML.load_file(File.open(File.expand_path(File.dirname(__FILE__)) + "/vagrant-config.yml"))


# # Clone ansible-bootstrap repository
# system("
#   if [ #{ARGV[0]} = 'up' ]; then
#     test -d roles || mkdir -p roles
#     if [ ! -d roles/ansible-debops ]; then
#       echo 'Cloning ansible-debops role'
#       git clone https://github.com/debops/ansible-debops roles/ansible-debops
#     else
#       echo 'Updating ansible-debops role'
#       cd roles/ansible-debops && git pull ; cd - >/dev/null
#     fi
#   fi
# ")

Vagrant.configure(2) do |config|
  config_yml[:vms].each do |name, settings|
    # use the config key as the vm identifier
    config.vm.define(name) do |vm_config|
      config.ssh.insert_key = false
      vm_config.vm.usable_port_range = (2200..2250)

      # This will be applied to all vms

      # Ubuntu
      vm_config.vm.box = "bento/ubuntu-16.04"

      # Get's honored normally
      vm_config.vm.synced_folder ".", "/vagrant", disabled: true
      # But not the centos box
      vm_config.vm.synced_folder '.', '/home/vagrant/sync', disabled: true

      # assign an ip address in the hosts network
      vm_config.vm.network "private_network", ip: settings[:ip]

      vm_config.vm.hostname = settings[:hostname]

      config.vm.provider "virtualbox" do |v|
        # make sure that the name makes sense when seen in the vbox GUI
        v.name = settings[:hostname]

        # Be nice to our users.
        v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
        v.customize ["modifyvm", :id, "--memory", settings[:ram], "--cpus", settings[:cpu]]
        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        # v.customize ["modifyvm", :id, "--natdnshostresolver2", "on"]
      end

      vm_config.vm.provision :ansible do |ansible|
        ansible.host_key_checking	= "false"
        # Disable default limit to connect to all the machines
        ansible.limit = "all"
        ansible.playbook = "playbook.yml"
        ansible.groups = config_yml[:groups]
        ansible.extra_vars = {
          deploy_env: "vagrant"
        }
      end
    end
  end
end
