# -*- mode: ruby -*-
# # vi: set ft=ruby :

# To skip Docker pre-load of calico/node and busybox, run vagrant up with:
#    vagrant up --provision-with file,shell

# The version of the calico docker image to install.  This is used to pre-load
# the calico/node image which slows down the install process, but speeds up the
# demonstration.
#
# This version should match the version required by calicotl installed in the
# cloud config files.
calico_docker_ver = "latest"

# Size of the cluster created by Vagrant
num_instances=1

# Change basename of the VM
instance_name_prefix="calico"

# Official CoreOS channel from which updates should be downloaded
update_channel='alpha'

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false

  config.vm.box = "coreos-%s" % update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % update_channel

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  # Set up each box
  (1..num_instances).each do |i|
    vm_name = "%s-%02d" % [instance_name_prefix, i]
    config.vm.define vm_name do |host|
      host.vm.hostname = vm_name

      ip = "172.18.18.#{i+100}"
      host.vm.network :private_network, ip: ip

      # Use a different cloud-init on the first server.
      if i == 1
        host.vm.provision :file, :source => "openssl/ca.pem", :destination => "/tmp/ca.pem"
        host.vm.provision :file, :source => "openssl/apiserver.pem", :destination => "/tmp/apiserver.pem"
        host.vm.provision :file, :source => "openssl/apiserver-key.pem", :destination => "/tmp/apiserver-key.pem"
        host.vm.provision :shell, :inline => "mkdir -p /etc/kubernetes/ssl", :privileged => true
        host.vm.provision :shell, :inline => "mv -t /etc/kubernetes/ssl /tmp/*.pem", :privileged => true
        host.vm.provision :shell, :inline => "chmod 600 /etc/kubernetes/ssl/apiserver-key.pem", :privileged => true
        host.vm.provision :shell, :inline => "chown root:root /etc/kubernetes/ssl/apiserver-key.pem", :privileged => true

        host.vm.provision :file, :source => "master-config-template.yaml", :destination => "/tmp/vagrantfile-user-data"
        host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      else
        host.vm.provision :file, :source => "node-config-template.yaml", :destination => "/tmp/vagrantfile-user-data"
        host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end
    end
  end
end
