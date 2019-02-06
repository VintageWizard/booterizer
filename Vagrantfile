# booterizer
# vagrant configuration
# LICENSE: MIT
require 'yaml'

#################################################################
# This is a hard fork of booterizer that ONLY supports IRIX 5.3 #
#################################################################
# Change settings in 'settings.yml' to match your environment   #
#################################################################
settings =      YAML.load_file 'settings.yml'

# irixversion install 5.3
irixversion =   settings['booterizer']['irixversion']
# installmethod can be via CD images or remote fetch from Internet
installmethod = settings['booterizer']['installmethod']
installmirror = settings['booterizer']['installmirror']

# your SGI box's hostname
clientname =    settings['booterizer']['clientname']
# whatever domain that you make up
clientdomain =  settings['booterizer']['clientdomain']

# Internal network your SGI, Virtual Machine will be on
network =       settings['booterizer']['network']
# Internal network's netmask
netmask =       settings['booterizer']['netmask']
# your virtual machine (bootp/tftp server]) will get this IP
hostip =        settings['booterizer']['hostip']
# your SGI box's IP address that you make up and set in prom via setenv netaddr
clientip =      settings['booterizer']['clientip']
# your SGI box's physical hardware address, from printenv at PROM - or eaddr for older PROM
clientether =   settings['booterizer']['clientether']

# This is the name of the interface on your physical machine that's connected to your SGI box
#   In my case, it's the ethernet adapter, which is en0 
bridgenic = settings['booterizer']['bridgenic']

# Your user account for your SGI box
myuser =        settings['booterizer']['myuser']
# Your desired shell 
  # if this doesn't exist, it defaults to /bin/csh
myshell =       settings['booterizer']['myshell']

# FTP urls

  ftpurls = YAML.load_file 'irix.5.3.yml'

foundation_baseurl    = ftpurls['ftpurls']['foundation']['baseurl']
foundation_disc1      = ftpurls['ftpurls']['foundation']['disc1']
devel_baseurl         = ftpurls['ftpurls']['devel']['baseurl']
devel_devfoundations  = ftpurls['ftpurls']['devel']['devfoundations']

#################################################################
# end of settings
#################################################################




current_dir = File.dirname(File.expand_path(__FILE__))     
disk_prefix = 'installdisk'
disk_ext ='.vdi'      
installdisk = "%s/%s%s" % [current_dir,disk_prefix,disk_ext] 

Vagrant.configure("2") do |config|
  config.vm.box = "debian/contrib-jessie64"
  config.vm.box_version = "8.11.0"
  #config.vm.network "public_network"
  config.vm.post_up_message = [ "booterizer configuration stage" ]
  
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  # Create XFS-formatted disk for extracted CD images
  config.vm.provider "virtualbox" do |v|
    unless File.exist?(installdisk)
      v.customize ['createhd', '--filename', installdisk, '--size', 50 * 1024]
    end
    v.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', installdisk]
  end
end

# Provision the new box with Ansible plays
Vagrant.configure("2") do |config|
  if Vagrant::Util::Platform.windows?
   config.vm.provision :shell do |shell|
      shell.inline = "sudo apt-get -y install wget curl"
    end 
    config.vm.provision :guest_ansible do |ansible|
      ansible.verbose = "v"
    ansible.playbook = "ansible/irix_ansible_setup.yml"
    ansible.extra_vars = {
        installmethod: installmethod,
        irixversion: irixversion,
        installmirror: installmirror,
        clientname: clientname,
        clientdomain: clientdomain,
        clientip: clientip,
        clientether: clientether,
        netmask: netmask,
        network: network,
        hostip: hostip,
        myuser: myuser,
        myshell: myshell,
        current_dir: current_dir,
        foundation_baseurl: foundation_baseurl,
        foundation_disc1: foundation_disc1,
        devel_baseurl: devel_baseurl,
        devel_devfoundations: devel_devfoundations
    }
    end
  else
    config.vm.provision "ansible" do |ansible|
      ansible.verbose = "v"
    ansible.playbook = "ansible/irix_ansible_setup.yml"
    ansible.extra_vars = {
        installmethod: installmethod,
        irixversion: irixversion,
        installmirror: installmirror,
        clientname: clientname,
        clientdomain: clientdomain,
        clientip: clientip,
        clientether: clientether,
        netmask: netmask,
        network: network,
        hostip: hostip,
        myuser: myuser,
        myshell: myshell,
        current_dir: current_dir,
        foundation_baseurl: foundation_baseurl,
        foundation_disc1: foundation_disc1,
        devel_baseurl: devel_baseurl,
        devel_devfoundations: devel_devfoundations,
        }    
    end
  end
end


Vagrant.configure("2") do |config|
  config.vm.box = "debian/contrib-jessie64"
  config.vm.box_version = "8.11.0"
  config.vm.network "public_network", ip: hostip, bridge: bridgenic
  config.vm.post_up_message = [ "booterizer running at ", hostip ]
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
end
