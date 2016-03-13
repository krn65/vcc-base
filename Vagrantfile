# -*- mode: ruby -*-
# vi: set ft=ruby :

# http://docs.ansible.com/ansible/guide_vagrant.html
Vagrant.require_version ">= 1.8.0"
Vagrant.configure(2) do |config|
  config.ssh.insert_key = false

  config.vm.define :dev_server do |node|
    node.vm.box = "ubuntu/trusty64"

    node.vm.provider :virtualbox do |vb|
      vb.memory = "2048"
      vb.cpus = "1"
    end

    node.vm.provision :ansible do |ansible|
      ansible.playbook = "site.yml"
    end
  end
end
