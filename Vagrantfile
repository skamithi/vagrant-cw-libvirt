# -*- mode: ruby -*-
# vi: set ft=ruby :
#
#
require 'json'

### CHECK THIS CONFIG BEFORE RUNNING VAGRANT UP
### RUN vagrant box list AND CHECK BOX NAMES ARE CORRECT
### THIS VAGRANTFILE ONLY WORKS WITH THE LIBVIRT PROVIDER
### CONFIGURE BOX NAMES
server_box_name = "trusty64"
cumulus_box_name = "cumulus.253"
###################################################

# udp port mappings. Messy. Can be simplified
# with some graph action..maybe using ruby-graphviz?

sp1_swp17_sp2 = [8000, 9000]
sp1_swp18_sp2 = [8001, 9001]
sp1_swp49_leaf1 = [8002, 9002]
sp1_swp50_leaf1 = [8003, 9003]
sp1_swp51_leaf2 = [8005, 9005]
sp1_swp52_leaf2 = [8006, 9006]
sp2_swp49_leaf2 = [8017, 9017]
sp2_swp50_leaf2 = [8018, 9018]
sp2_swp51_leaf1 = [8007, 9007]
sp2_swp52_leaf1 = [8008, 9008]
sp2_swp50_leaf2 = [8009, 9009]
#sp2_swp51_leaf2 = [8010, 9010]
leaf1_swp17_leaf2 = [8011, 9011]
leaf1_swp18_leaf2 = [8012, 9012]
leaf1_swp32s0_svr1 = [8013, 9013]
leaf1_swp32s1_svr2 = [8014, 9014]
leaf2_swp32s0_svr2 = [8015, 9015]
leaf2_swp32s1_svr1 = [8016, 9016]

wbench_hostlist = [:spine1, :spine2, :leaf1, :leaf2, :server1, :server2]
last_ip_octet = 100
last_mac_octet = 11
wbench_hosts = { :wbench_hosts => {} }


wbench_hostlist.each do |hostentry|
  wbench_hosts[:wbench_hosts][hostentry] = {
    :ip => '192.168.0.' + last_ip_octet.to_s,
    :mac => '12:11:22:33:44:' + last_mac_octet.to_s
  }
  last_mac_octet += 1
  last_ip_octet += 1
end

Vagrant.configure(2) do |config|

  # increase nic adapter count to be greater than 8
  # for all VMs.
  config.vm.provider :libvirt do |domain|
    domain.nic_adapter_count = 20
  end

  config.vm.box = cumulus_box_name
  # vagrant issues #1673..fixes hang with configure_networks
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.define :wbenchvm do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 1024
    end
    node.vm.box = server_box_name
    # disabling sync folder support on all VMs
    node.vm.synced_folder '.', '/vagrant', :disabled => true

    # wbench_eth1
    node.vm.network :private_network,
      :ip => '192.168.0.1/24',
      :libvirt__forward_mode => 'veryisolated',
      :libvirt__dhcp_enabled => false,
      :libvirt__network_name => 'switch_mgmt'
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'ccw-wbenchvm-ansible/site.yml'
      ansible.extra_vars = wbench_hosts
    end
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'playbooks/wbenchvm_extra.yml'
    end
  end

  config.vm.define :spine1 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    # disabling sync folder support on all vms
    node.vm.hostname = 'spine1'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth0 (after deleting vagrant interface)
    node.vm.network :private_network,
        :auto_config => false,
        :libvirt__forward_mode => 'veryisolated',
        :libvirt__dhcp_enabled => false,
        :libvirt__network_name => 'switch_mgmt',
        :mac => wbench_hosts[:wbench_hosts][:spine1][:mac]
    # swp17 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp17_sp2[0],
      :libvirt__tunnel_local_port =>  sp1_swp17_sp2[1]
    # swp18 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp18_sp2[0],
      :libvirt__tunnel_local_port => sp1_swp18_sp2[1]
    # swp49 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp49_leaf1[0],
      :libvirt__tunnel_local_port => sp1_swp49_leaf1[1]

    # swp50 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp50_leaf1[0],
      :libvirt__tunnel_local_port => sp1_swp50_leaf1[1]

    # swp51 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp51_leaf2[0],
      :libvirt__tunnel_local_port => sp1_swp51_leaf2[1]
    # swp52 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp52_leaf2[0],
      :libvirt__tunnel_local_port => sp1_swp52_leaf2[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'playbooks/update_switches.yml'
    end
  end

  config.vm.define :spine2 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    # disabling sync folder support on all vms
    node.vm.hostname = 'spine2'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth0 (after deleting vagrant interface)
    node.vm.network :private_network,
        :auto_config => false,
        :libvirt__forward_mode => 'veryisolated',
        :libvirt__dhcp_enabled => false,
        :libvirt__network_name => 'switch_mgmt',
        :mac => wbench_hosts[:wbench_hosts][:spine2][:mac]

    # swp17 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp17_sp2[1],
      :libvirt__tunnel_local_port =>  sp1_swp17_sp2[0]
    # swp18 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp18_sp2[1],
      :libvirt__tunnel_local_port => sp1_swp18_sp2[0]
    # swp49 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp49_leaf2[0],
      :libvirt__tunnel_local_port => sp2_swp49_leaf2[1]

    # swp50 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp50_leaf2[0],
      :libvirt__tunnel_local_port => sp2_swp50_leaf2[1]

    # swp51 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp51_leaf1[0],
      :libvirt__tunnel_local_port => sp2_swp51_leaf1[1]
    # swp52 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp52_leaf1[0],
      :libvirt__tunnel_local_port => sp2_swp52_leaf1[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'playbooks/update_switches.yml'
    end
  end

  config.vm.define :leaf1 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    # disabling sync folder support on all vms
    node.vm.hostname = 'leaf1'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth0 (after deleting vagrant interface)
    node.vm.network :private_network,
        :auto_config => false,
        :libvirt__forward_mode => 'veryisolated',
        :libvirt__dhcp_enabled => false,
        :libvirt__network_name => 'switch_mgmt',
        :mac => wbench_hosts[:wbench_hosts][:leaf1][:mac]
    # swp1s0 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp49_leaf1[1],
      :libvirt__tunnel_local_port => sp1_swp49_leaf1[0]
    # swp1s1 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp50_leaf1[1],
      :libvirt__tunnel_local_port => sp1_swp50_leaf1[0]
    # swp1s2 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp51_leaf1[1],
      :libvirt__tunnel_local_port => sp2_swp51_leaf1[0]

    # swps1s3 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp52_leaf1[1],
      :libvirt__tunnel_local_port => sp2_swp52_leaf1[0]

    # swp17 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp17_leaf2[0],
      :libvirt__tunnel_local_port => leaf1_swp17_leaf2[1]
    # swp18 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp18_leaf2[0],
      :libvirt__tunnel_local_port => leaf1_swp18_leaf2[1]
    # swp32s0 (eth7)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp32s0_svr1[0],
      :libvirt__tunnel_local_port => leaf1_swp32s0_svr1[1]
    # swp32s1 (eth8)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp32s1_svr2[0],
      :libvirt__tunnel_local_port => leaf1_swp32s1_svr2[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'playbooks/update_switches.yml'
    end
  end

  config.vm.define :leaf2 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    # disabling sync folder support on all vms
    node.vm.hostname = 'leaf2'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth0 (after deleting vagrant interface)
    node.vm.network :private_network,
        :auto_config => false,
        :libvirt__forward_mode => 'veryisolated',
        :libvirt__dhcp_enabled => false,
        :libvirt__network_name => 'switch_mgmt',
        :mac => wbench_hosts[:wbench_hosts][:leaf2][:mac]
    # swp1s0 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp49_leaf2[1],
      :libvirt__tunnel_local_port => sp2_swp49_leaf2[0]
    # swp1s1 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp50_leaf2[1],
      :libvirt__tunnel_local_port => sp2_swp50_leaf2[0]
    # swp1s2 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp51_leaf2[1],
      :libvirt__tunnel_local_port => sp1_swp51_leaf2[0]
    # swps1s3 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp52_leaf2[1],
      :libvirt__tunnel_local_port => sp1_swp52_leaf2[0]

    # swp17 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp17_leaf2[1],
      :libvirt__tunnel_local_port => leaf1_swp17_leaf2[0]
    # swp18 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp18_leaf2[1],
      :libvirt__tunnel_local_port => leaf1_swp18_leaf2[0]
    # swp32s0 (eth7)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s0_svr2[0],
      :libvirt__tunnel_local_port => leaf2_swp32s0_svr2[1]
    # swp32s1 (eth8)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s1_svr1[0],
      :libvirt__tunnel_local_port => leaf2_swp32s1_svr1[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'playbooks/update_switches.yml'
    end
  end

  config.vm.define :server1 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    node.vm.box = server_box_name
    # disabling sync folder support on all vms
    node.vm.hostname = 'server1'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth0 (after deleting vagrant interface)
    node.vm.network :private_network,
        :auto_config => false,
        :libvirt__forward_mode => 'veryisolated',
        :libvirt__dhcp_enabled => false,
        :libvirt__network_name => 'switch_mgmt'
    # eth1
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s1_svr1[1],
      :libvirt__tunnel_local_port => leaf2_swp32s1_svr1[0]
    # eth2
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp32s0_svr1[1],
      :libvirt__tunnel_local_port => leaf1_swp32s0_svr1[0]
  end

  config.vm.define :server2 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    node.vm.box = server_box_name
    # disabling sync folder support on all vms
    node.vm.hostname = 'server2'
    node.vm.synced_folder '.', '/vagrant', :disabled => true
    # eth0 (after deleting vagrant interface)
    node.vm.network :private_network,
        :auto_config => false,
        :libvirt__forward_mode => 'veryisolated',
        :libvirt__dhcp_enabled => false,
        :libvirt__network_name => 'switch_mgmt'
    # eth1
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp32s1_svr2[1],
      :libvirt__tunnel_local_port => leaf1_swp32s1_svr2[0]
    # eth2
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s0_svr2[1],
      :libvirt__tunnel_local_port => leaf2_swp32s0_svr2[0]
  end
end
