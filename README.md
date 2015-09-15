# Recreating Cumulus Workbench like environment using libvirt-vagrant

**(not supported by Cumulus)**

## Requirements

* Ubuntu 14.04 on Bare metal server
* patched [libvirt with udp unicast
  support](https://launchpad.net/~linuxsimba/+archive/ubuntu/libvirt-udp-tunnel)
* install latest [libvirt vagrant plugin](http://linuxsimba.com/vagrant-libvirt-install/) from its master branch.

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
```

After installation, the leaf and spine switches cannot be controlled using
vagrant up because the vagrant mgmt interface is deleted. To manage bringing up
the spine and leaf switches, use ``virt-manager`` or ``virsh start``.

Really wish vagrant supported vagrant up without requiring a working vagrant
mgmt interface.

## TODO

* include stubs for cl-license and switchd so that cldemo playbooks work without
  any modification

## LICENSE
MIT





