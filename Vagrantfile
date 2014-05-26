# -*- mode: ruby -*-
# vi: set ft=ruby sw=2 sts=2 ts=2:

VAGRANTFILE_API_VERSION = "2"

$rankmirros = <<SCRIPT
(
  rankmirrors /etc/pacman.d/mirrorlist > /etc/pacman.d/new &&
  mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.pacsave &&
  mv /etc/pacman.d/new /etc/pacman.d/mirrorlist
)
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # the vm must have the interfaces name properly configure with the udev,
  # using ethx and not the new naming convention enxxx otherwise private
  # network won't work
  config.vm.box = "archlinux-64"
  config.vm.box_url = "http://cloud.terry.im/vagrant/archlinux-x86_64.box"
  config.vm.box_download_checksum = "1afc845cd491eee8877ebd31e7293023c2186cec0068cc72014569edb607df19"
  config.vm.box_download_checksum_type = "sha256"

  config.ssh.forward_agent = true
  # config.vm.network "public_network"
  # config.vm.synced_folder "../data", "/vagrant_data"

  # config.vm.provider "virtualbox" do |vb|
  #   vb.gui = true
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  
  config.vm.provision "shell" do |shell|
    shell.inline = $rankmirros
    shell.inline = "pacman -Syy && pacman -S python2 --noconfirm"
  end

  require 'yaml'
  inventory = YAML.load_file('inventory.yaml')

  ansible_extra_vars = {
    "app_name" => "app",
    "user" => "vagrant",
    "ansible_python_interpreter" => "/usr/bin/python2",
    "project_root" => "/vagrant",
  }

  inventory.each do |server|
    config.vm.define server["name"] do |instance|
      # instance.vm.network "private_network", ip: server["ip"]

      instance.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/playbook.yaml"
        ansible.verbose = "vv"
        ansible.extra_vars = ansible_extra_vars

        server["groups"].each do |group| 
          if not ansible.groups.is_a?(Hash)
            ansible.groups = {
              group => [server["name"]]
            }
          elsif
            group_list = ansible.groups[group] ||= []
            group_list.push(server["name"])
          end
        end
      end
    end
  end
end
