# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'fileutils'
CLOUD_CONFIG_PATH = File.join(File.dirname(__FILE__), "cloud-config")
CLOUD_CONFIG_TMP_PATH = "/tmp/vagrant-cloud-config"

def getCloudConfig(cloud_config, config)
   data = File.read(CLOUD_CONFIG_PATH) 
   cloud_config.each do |key, value|
      data = data.gsub("$#{key}", value) 
   end

   File.open(CLOUD_CONFIG_TMP_PATH, "w") do |f|
      f.write(data)
   end

   return data
end

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    ## GENERIC
    ## USE: https://github.com/mkaczanowski/vagrant-vultr
    config.vm.provider :vultr do |vultr, override|
        override.ssh.private_key_path = '~/.ssh/vultr'
        override.vm.box_url = 'https://github.com/p0deje/vagrant-vultr/raw/master/box/vultr.box'
    
        vultr.token = 'VULTR_API_KEY'
        vultr.region = 'New Jersey'
        vultr.plan = '768 MB RAM,15 GB SSD,1.00 TB BW'
        vultr.os = 'CoreOS Stable'
        vultr.enable_ipv6 = 'yes'
        vultr.enable_private_network = 'no'

        if File.exist?(CLOUD_CONFIG_PATH)
          override.vm.provision :shell, :inline => "rm -rf /root/configs", :privileged => true
          override.vm.provision :file, :source => "priv/openvpn", :destination => "/root/configs/openvpn"
          override.vm.provision :file, :source => "#{CLOUD_CONFIG_TMP_PATH}", :destination => "/tmp/vagrantfile-user-data"
          override.vm.provision :shell, :inline => 
            "mkdir -p /var/lib/coreos-vagrant/ && mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/",
            :privileged => true
          override.vm.provision :shell, :inline =>
            "coreos-cloudinit -from-file /var/lib/coreos-vagrant/vagrantfile-user-data",
            :privileged => true
        end
    end

    ## VMS
    config.vm.define "us_tunnel" do |box|
        box.vm.box = 'us_tunnel'

        config.vm.provider :vultr do |vultr, override|
            vultr.hostname = box.vm.box
            vultr.label = box.vm.box
        end

        cloud_config = {
            :hostname => box.vm.box,
            :no_ip_username => "EMAIL:KEY",
            :no_ip_hostname => "DDNS_HOSTNAME",
            :no_ip_allowed_sources => "WHITELISTED_SOURCE_ADDRS",
        }
        getCloudConfig(cloud_config, config)
    end 
end
