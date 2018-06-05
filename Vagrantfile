# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 2.0.0"

required_plugins = %w(vagrant-cachier)

plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
if not plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort "Installation of one or more plugins has failed. Aborting."
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.box_check_update = false
  config.vbguest.auto_update = false

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
    config.cache.auto_detect = false
    config.cache.enable :yum
  end


  config.vm.define :apachefun, primary: true do |apachefun|
    apachefun.vm.provision "shell", path: "configure-box.sh"
    apachefun.vm.provider "virtualbox" do |vb|
      vb.name = "apache-ansible"
      vb.memory = 2048
      vb.cpus = 2
    end


    apachefun.vm.hostname = "vagrant.apache.fun.dis.nz"
    apachefun.vm.network :private_network, ip: "10.128.250.2", netmask: "255.0.0.0", auto_config: false


    apachefun.vm.provision "ansible_local", type: "ansible_local" do |ansible|
      ansible.install            = true
      ansible.compatibility_mode = "2.0"
      ansible.install_mode       = "default"
      ansible.playbook           = "site.yml"
      ansible.limit              = "all"
      ansible.inventory_path     = "staging/vagrant"
      ansible.galaxy_role_file   = "roles/requirements.yml"
      ansible.galaxy_command     = "sudo ansible-galaxy install --role-file=%{role_file} -p /vagrant/roles" # download as local role, only once

    end

  end


end
