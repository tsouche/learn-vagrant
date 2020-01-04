# -*- mode: ruby -*-
# vi: set ft=ruby :

# This vagrant file will setup 3 VMs based on Ubuntu server, to deploy a
# Kubernetes cluster (1 master ad 2 slaves)


config_files_path="./source"

# Generate the token needed for a node to join the cluster
require 'securerandom'
random_string1 = SecureRandom.hex
random_string2 = SecureRandom.hex
cluster_init_token = "#{random_string1[0..5]}.#{random_string2[0..15]}"


# Set global variables
NUM_NODES = 2           # number of Slave nodes to be generated
MASTER_CPU = 2          # Master's CPUs number
MASTER_MEM = 4096       # Master's memory size
SLAVE_CPU = 2           # Slaves' CPUs number
SLAVE_MEM = 2048        # Slaves' memory size
START_IP = "192.168.0.20"

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  # Build the base from a clean Ubuntu 18.04 server
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.box_version = "201912.03.0"
  config.vm.box_check_update = false
  config.vm.provision :shell, path: "./source/k8s_bootstrap_shared_base.sh"

  # Configure the Master node
  config.vm.define "k8s-master", primary: true do |master|
    # set the machine name
    master.vm.hostname = "k8s-master"
    # set the private ip@
    master_ip = "#{START_IP}0"
    master.vm.network 'public_network', ip: "#{master_ip}"
    # dimension the VM
    master.vm.provider "virtualbox" do |vb|
      vb.cpus = MASTER_CPU
      vb.memory = MASTER_MEM
      #vb.customize ["modifyvm", :id, "--cpuexecutioncap", "70"]
      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
    # apply the master specific script
    master.vm.provision :shell, path: "./source/k8s_bootstrap_master.sh"
  end

  # Configure the Slave nodes
  (1..NUM_NODES).each do |slave_number|

    # set the machine name
    slave_name = "k8s-slave#{slave_number}"
    # configure each slave machine
    config.vm.define slave_name do |node|
      # set the machine name
      node.vm.hostname = "#{slave_name}"
      # set the private ip@
      slave_address = slave_number
      slave_ip = "#{START_IP}#{slave_number}"
      node.vm.network 'public_network', ip: "#{slave_ip}"
      # dimension the VM
      node.vm.provider 'virtualbox' do |vb|
        vb.cpus = SLAVE_CPU
        vb.memory = SLAVE_MEM
        #vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
        vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
      end
      # apply the master specific script
      node.vm.provision :shell, path: "./source/k8s_bootstrap_slave.sh"
    end
  end

  # Cleanup temporary files
  # config.vm.provision :shell, path: "./source/k8s_cleanup.sh"

end
