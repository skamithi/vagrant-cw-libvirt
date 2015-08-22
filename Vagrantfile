# -*- mode: ruby -*-
# vi: set ft=ruby :
#
#
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
sp2_swp51_leaf2 = [8010, 9010]
leaf1_swp17_leaf2 = [8011, 9011]
leaf1_swp18_leaf2 = [8012, 9012]
leaf1_swp32s0_svr1 = [8013, 9013]
leaf1_swp32s1_svr2 = [8014, 9014]
leaf2_swp32s0_svr2 = [8015, 9015]
leaf2_swp32s1_svr1 = [8016, 9016]
vm_mac = {
  'spine1' => '12:11:22:33:44:55',
  'spine2' => '12:11:22:33:44:66',
  'leaf1' =>  '12:11:22:33:44:77',
  'leaf2' =>  '12:11:22:33:44:88'
}
Vagrant.configure(2) do |config|

  # increase nic adapter count to be greater than 8
  # for all VMs.
  config.vm.provider :libvirt do |domain|
    domain.nic_adapter_count = 20
  end

  config.vm.box = 'cumulus.253'
  # vagrant issues #1673..fixes hang with configure_networks
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  config.vm.define :wbenchvm do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    node.vm.box = "trusty64_4"
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
        :mac => vm_mac['spine1']
    # swp17 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp17_sp2[0],
      :libvirt__tunnel_source_port =>  sp1_swp17_sp2[1]
    # swp18 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp18_sp2[0],
      :libvirt__tunnel_source_port => sp1_swp18_sp2[1]
    # swp49 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp49_leaf1[0],
      :libvirt__tunnel_source_port => sp1_swp49_leaf1[1]

    # swp50 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp50_leaf1[0],
      :libvirt__tunnel_source_port => sp1_swp50_leaf1[1]

    # swp51 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp51_leaf2[0],
      :libvirt__tunnel_source_port => sp1_swp51_leaf2[1]
    # swp52 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp52_leaf2[0],
      :libvirt__tunnel_source_port => sp1_swp52_leaf2[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'update_switches.yml'
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
        :mac => vm_mac['spine2']
    # swp17 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp17_sp2[1],
      :libvirt__tunnel_source_port =>  sp1_swp17_sp2[0]
    # swp18 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp18_sp2[1],
      :libvirt__tunnel_source_port => sp1_swp18_sp2[0]
    # swp49 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp49_leaf2[0],
      :libvirt__tunnel_source_port => sp2_swp49_leaf2[1]

    # swp50 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp50_leaf2[0],
      :libvirt__tunnel_source_port => sp2_swp50_leaf2[1]

    # swp51 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp51_leaf1[0],
      :libvirt__tunnel_source_port => sp2_swp51_leaf1[1]
    # swp52 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp52_leaf1[0],
      :libvirt__tunnel_source_port => sp2_swp52_leaf1[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'update_switches.yml'
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
        :mac => vm_mac['leaf1']
    # swp1s0 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp49_leaf1[1],
      :libvirt__tunnel_source_port => sp1_swp49_leaf1[0]
    # swp1s1 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp50_leaf1[1],
      :libvirt__tunnel_source_port => sp1_swp50_leaf1[0]
    # swp1s2 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp51_leaf1[1],
      :libvirt__tunnel_source_port => sp2_swp51_leaf1[0]

    # swps1s3 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp52_leaf1[1],
      :libvirt__tunnel_source_port => sp2_swp52_leaf1[0]

    # swp17 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp17_leaf2[0],
      :libvirt__tunnel_source_port => leaf1_swp17_leaf2[1]
    # swp18 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp18_leaf2[0],
      :libvirt__tunnel_source_port => leaf1_swp18_leaf2[1]
    # swp32s0 (eth7)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp32s0_svr1[0],
      :libvirt__tunnel_source_port => leaf1_swp32s0_svr1[1]
    # swp32s1 (eth8)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp32s1_svr2[0],
      :libvirt__tunnel_source_port => leaf1_swp32s1_svr2[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'update_switches.yml'
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
        :mac => vm_mac['leaf2']
    # swp1s0 (eth1)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp49_leaf2[1],
      :libvirt__tunnel_source_port => sp2_swp49_leaf2[0]
    # swp1s1 (eth2)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp2_swp50_leaf2[1],
      :libvirt__tunnel_source_port => sp2_swp50_leaf2[0]
    # swp1s2 (eth3)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp51_leaf2[1],
      :libvirt__tunnel_source_port => sp1_swp51_leaf2[0]
    # swps1s3 (eth4)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => sp1_swp52_leaf2[1],
      :libvirt__tunnel_source_port => sp1_swp52_leaf2[0]

    # swp17 (eth5)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp17_leaf2[1],
      :libvirt__tunnel_source_port => leaf1_swp17_leaf2[0]
    # swp18 (eth6)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf1_swp18_leaf2[1],
      :libvirt__tunnel_source_port => leaf1_swp18_leaf2[0]
    # swp32s0 (eth7)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s0_svr2[0],
      :libvirt__tunnel_source_port => leaf2_swp32s0_svr2[1]
    # swp32s1 (eth8)
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s1_svr1[0],
      :libvirt__tunnel_source_port => leaf2_swp32s1_svr1[1]
    node.vm.provision :ansible do |ansible|
      ansible.playbook = 'update_switches.yml'
    end
  end

  config.vm.define :server1 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    node.vm.box = "trusty64_4"
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
      :libvirt__tunnel_port => leaf1_swp32s0_svr1[1],
      :libvirt__tunnel_source_port => leaf1_swp32s0_svr1[0]
    # eth2
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s1_svr1[1],
      :libvirt__tunnel_source_port => leaf2_swp32s1_svr1[0]
  end

  config.vm.define :server2 do |node|
    node.vm.provider :libvirt do |domain|
      domain.memory = 256
    end
    node.vm.box = "trusty64_4"
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
      :libvirt__tunnel_source_port => leaf1_swp32s1_svr2[0]
    # eth2
    node.vm.network :private_network,
      :libvirt__tunnel_type => 'udp',
      :libvirt__tunnel_port => leaf2_swp32s0_svr2[1],
      :libvirt__tunnel_source_port => leaf2_swp32s0_svr2[0]
  end
end
