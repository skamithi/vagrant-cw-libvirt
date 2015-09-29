# Recreating Cumulus Workbench like environment using libvirt-vagrant

**(not supported by Cumulus)**

## Requirements

* Ubuntu 14.04 on Bare metal laptop/server
* patched [libvirt with udp unicast support](https://launchpad.net/~linuxsimba/+archive/ubuntu/libvirt-udp-tunnel)
* libvirt-vagrant (> 0.0.31 - Available as of Sept 2015 on Rubygems)
* Vagrant (of cause!)

## Topology
Reproduces a 2 spine/2 leaf/2 server cumulus workbench architecture
![2 spine 2 leaf](https://support.cumulusnetworks.com/hc/en-us/article_attachments/201247418/ibgp-2lt22s.png)


## Installation

```
$ git clone https://github.com/skamithi/vagrant-cw-libvirt
$ cd vagrant-cw-libvirt
$ git submodule init
$ git submodule update
$ vagrant box add http://linuxsimba.com/vagrantbox/ubuntu-trusty.box --name trusty64
$ vagrant box add http://linuxsimba.com/vagrantbox/cumulus-253.box --name cumulus.253
$ vagrant up --no-parallel
$ vagrant ssh wbenchvm
```

This should drop you straight into the wbenchvm as the cumulus user. For now
only Ansible and puppet cldemos work.

By default logging in as root to any other device in the topology from the wbenchvm, should just work.
SSH key from wbenchvm cumulus user is copied to all managed VMs during the
vagrant provisioning process. This is why the wbenchvm must be provisioned
first.

After installation, the leaf and spine switches cannot be controlled using
vagrant up because the vagrant mgmt interface is deleted. To manage bringing up
the spine and leaf switches, use ``virt-manager`` or ``virsh start``.


Really wish vagrant supported vagrant up without requiring a working vagrant
mgmt interface.

## TODO

* Write ZTP reset scripts.
* Write script to simulate ``vagrant up`` on switches after switches lose their
Vagrant managed interface.

## LICENSE
MIT





