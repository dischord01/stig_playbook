# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # PROXY
  # vagrant box add --insecure hashicorp/centos/7
  # vagrant plugin install --plugin-source http://rubygems.org vagrant-proxyconf
  # config.proxy.http     = "http://something.org:80"
  # config.proxy.https    = "http://something.org:80"
  # config.proxy.no_proxy = "localhost,127.0.0.1,.example.com"

  # Default Hashi corp CentOS 7 box.
  config.vm.box = "rhel.6.5"

  # Fixes changes from https://github.com/mitchellh/vagrant/pull/4707
  config.ssh.insert_key = false

  config.vm.provider :virtualbox do |vb|
    host = RbConfig::CONFIG['host_os']
    # Give VM 1/4 system memory 
    if host =~ /darwin/
      # sysctl returns Bytes and we need to convert to MB
      mem = `sysctl -n hw.memsize`.to_i / 1024
    elsif host =~ /linux/
      # meminfo shows KB and we need to convert to MB
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i
    elsif host =~ /mswin|mingw|cygwin/
      # Windows code via https://github.com/rdsubhas/vagrant-faster
      mem = `wmic computersystem Get TotalPhysicalMemory`.split[1].to_i / 1024
    end

    mem = mem / 1024 / 4
    vb.customize ["modifyvm", :id, "--memory", mem]
    # vb.customize ["modifyvm", :id, "--memory", 3072] # RAM allocated to each VM
    vb.gui = false
    vb.cpus = 1
  end

  # Red Hat Registration
  # vagrant plugin install vagrant-registration
  # if Vagrant.has_plugin?('vagrant-registration')
  #   config.registration.username = ''
  #   config.registration.password = ''
  #   config.registration.pools    = ''
  #   config.registration.unregister_on_halt = false 
  # end

  # NFS
  # sudo yum -y install nfs-utils nfs-utils-lib portmap

  # Ansible 
  config.vm.provision "ansible" do |ansible|
    ansible.inventory_path = "inventory"
    ansible.limit = "all"
    ansible.playbook = "site.yml"
  end

  # RHEL 7.2
  config.vm.define :rhel6 do |rhel6|
    rhel6.vm.hostname = "rhel6.local"
    rhel6.vm.network :private_network, ip: "192.168.60.101"
    # rhel6.vm.network :forwarded_port, guest: 8443, host: 8443
    rhel6.vm.synced_folder ".", "/vagrant", nfs: true #type: "nfs" 
  end



end

