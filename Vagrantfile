# -*- mode: ruby -*-
# vi: set ft=ruby :

# load configs
require 'yaml'
current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/config.yml")
vagrant_config = configs['configs']

# require necessary plugins
# required_plugins = %w( vagrant-hostmanager vagrant-vbguest )
# required_plugins.each do |plugin|
#   system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
# end

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  # config.vm.synced_folder "ansible_vagrant", "/vagrant/ansible_vagrant", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]
  config.ssh.insert_key = false
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true

  # Define four VMs with static private IP addresses.
  vmboxes = [
    { 
      :name   => "master_1", 
      :ip     => vagrant_config['master_1']['ip'], 
      :memory => vagrant_config['master_1']['memory'],
      :cpus   => vagrant_config['master_1']['cpus'],
      :domain => vagrant_config['master_1']['domain']
    },
    { 
      :name   => "data_1", 
      :ip     => vagrant_config['data_1']['ip'], 
      :memory => vagrant_config['data_1']['memory'],
      :cpus   => vagrant_config['data_1']['cpus'],
      :domain => vagrant_config['data_1']['domain']
    },
    { 
      :name   => "data_2", 
      :ip     => vagrant_config['data_2']['ip'], 
      :memory => vagrant_config['data_2']['memory'],
      :cpus   => vagrant_config['data_2']['cpus'],
      :domain => vagrant_config['data_2']['domain']
    }
  ]

  # Provision each of the VMs.
  vmboxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.provider "virtualbox" do |vb|
        vb.memory = opts[:memory]
        vb.cpus = opts[:cpus]
        # linked_clone: https://www.vagrantup.com/docs/providers/virtualbox/configuration#linked-clones
        vb.linked_clone = true
        vb.customize ["modifyvm", :id, "--audio", "none"]
      end
      
      config.vbguest.auto_update = true
      config.vm.hostname = opts[:name]
      config.vm.network :private_network, ip: opts[:ip]
      config.vm.hostname = opts[:domain]
      # config.hostmanager.aliases = opts[:aliases]
      config.ssh.forward_agent = true
      config.ssh.forward_x11 = true

      # Provision all the VMs using Ansible after last VM is up.
      if opts[:name] == "data_2" # this condition to check if it's the last loop(last VM).
        # provision master-playbook on its hosts.
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible_vagrant/playbook.yml"
          # ansible.galaxy_role_file = "ansible_vagrant/requirements.yml"
          ansible.inventory_path = "ansible_vagrant/inventory/hosts"
          ansible.limit = "all"
        end
      end
    end
  end
end