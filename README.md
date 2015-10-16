# Recreating Cumulus Workbench like environment using libvirt-vagrant

**(not supported by Cumulus Networks)**


## Requirements

* Ubuntu 14.04 on Bare metal laptop/server
* patched [libvirt with udp unicast support](https://launchpad.net/~linuxsimba/+archive/ubuntu/libvirt-udp-tunnel)
* libvirt-vagrant (> 0.0.31 - Available as of Sept 2015 on Rubygems)
* cumulus vagrant plugin - see installation notes.
* Vagrant (of cause!)
* 40GB Available Disk Space
* 4GB of Available RAM.

## Topology
Reproduces a 2 spine/2 leaf/2 server cumulus workbench architecture
![2 spine 2 leaf](https://support.cumulusnetworks.com/hc/en-us/article_attachments/201247418/ibgp-2lt22s.png)


## Installation

Run the following steps to get the setup going. Its important to run ``vagrant up`` in a non parallel mode. Bringing up multiple virtual machines simultaneously in libvirt is brittle. So best to leave that feature off for now.
```
$ apt-get install libvirt-dev (recommended in case libvirt-vagrant needs to compile)
$ git clone https://github.com/skamithi/vagrant-cw-libvirt
$ cd vagrant-cw-libvirt
$ vagrant plugin install libvirt-vagrant
$ vagrant plugin install cumulus-vagrant
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

### Problems during install

####  libvirt cannot find its default storage pool

On ``vagrant up`` one may receive the following error:

```
There was error while creating libvirt storage pool: Call to
virStoragePoolDefineXML failed: operation failed: pool 'default' already exists
with uuid f888ea25-5315-06ea-4418-ddf216d46850
```

Run ``virsh pool --all`` to confirm if the default pool exists. Sometimes
when first installed it may be in an inactive state. In this case, just activate
it

```
$ virsh pool-list --all
 Name                 State      Autostart
-------------------------------------------
 default              inactive     no

```

To fix this, run the following commands

```
$ virsh pool-autostart default
$ virsh pool-start default
```




## EXAMPLE USING A CLDEMO

_demos are broken - Oct 16 2015. Fix/Review under investigation_

### Ansible OSPF Unnumbered Demo

* Log into the Cldemo environment. This logs you straight into the cumulus user instead of the vagrant user.
```
$ vagrant ssh wbenchvm
```

* Install Ansible OSPF demo

```
cumulus@wbench:~$ apt-cache search cldemo | grep ansible

cldemo-wbench-base-ansible - Ansible basic configuration
cldemo-wbench-hortonworks-2lt22s-ansible - Trident2 2 leaf 2 spine; Hosts
running Hortonworks HDP
cldemo-wbench-ibgp-ansible - Ansible, PTM and iBGP IPv4
cldemo-wbench-ospfunnum-ansible - Ansible, PTM and OSPF Unnumbered IPv4
cldemo-wbench-ospfunnum-base-ansible - Ansible, PTM and OSPF Unnumbered IPv4
cldemo-wbench-ztp-ansible - Zero touch provisioning script for Ansible

cumulus@wbench:~$ sudo apt-get install cldemo-wbench-ospfunnum-ansible

```

*  Get the switches to re-run ZTP. Got a script to do this.
```
cumulus@wbench:~$ $HOME/clear_ztp_all_switches.sh
```

* Run the Ansible OSPF unnumbered demo
```
cumulus@wbench:~$ cd $HOME/example-ospfunnum-ansible

cumulus@wbench:~/example-ospfunnum-ansible$ ansible-playbook  hosts-2lt22s -i
site-ospfunnum-2lt22s.yml
```

* Verify that the config occurred properly. Run the cldemo serverspec tests. Most tests should fail. The failed tests occur because it is checking for files found only on HW switches.
```
cumulus@wbench:~$ cd $HOME/cldemo-tests
cumulus@wbench:~/cldemo-tests$ bundle exec rake ospfunnum 2lt22s
...
.....
........
Test report summary

TARGET | TOTAL | PASSED | FAILED
-------|-------|--------|-------
leaf1  | 64    | 59     | 5
leaf2  | 64    | 59     | 5
spine1 | 58    | 53     | 5
spine2 | 58    | 53     | 5

```

* Log into a switch and investigate the switching and routing. Let's look at leaf1
```
# ssh root@leaf1
root@leaf1# apt-get install netshow
root@leaf1# netshow int
root@leaf1# cl-ospf show route
```
### Puppet OSPF Unnumbered Demo



* Log into the Cldemo environment. This logs you straight into the cumulus user instead of the vagrant user.
```
$ vagrant ssh wbenchvm
```

* If you have configuration on the switches and want to wipe it up, then log out of the vagrant environment and destroy the VMs and regenerate the VMs
```
cumulus@wbench:~$ exit
$ vagrant destroy leaf1 leaf2 spine1 spine2
$ vagrant up --no-parallel leaf1 leaf2 spine1 spine2
```

* Install Puppet OSPF demo

```
cumulus@wbench:~$ apt-cache search cldemo | grep puppet

cumulus@wbench:~$ apt-cache search cldemo | grep puppet
cldemo-wbench-base-puppet - Puppetmaster basic configuration
cldemo-wbench-ganglia-2lt22s-puppet - 2 leaf 2 spine; Puppet, OSPF Unnumbered IPv4, and Ganglia
cldemo-wbench-ganglia-2s-puppet - 2 switch; Puppet, OSPF Unnumbered IPv4, and Ganglia
cldemo-wbench-ibgp-puppet - Puppet, PTM and iBGP IPv4
cldemo-wbench-librenms-2lt22s-puppet - Technology specific puppet configuration files for LibreNMS installation
cldemo-wbench-librenms-2s-puppet - Technology specific puppet configuration files for LibreNMS installation
cldemo-wbench-librenms-base-puppet - Puppet configuration files for LibreNMS installation
cldemo-wbench-openstack-2lt22s-puppet - Openstack setup package for Puppet
cldemo-wbench-osinstaller-ubuntuserver-trusty-puppet - Installation of Ubuntu Server 14.04 with Puppet servers
cldemo-wbench-ospfunnum-base-puppet - Puppet, PTM and OSPF Unnumbered IPv4
cldemo-wbench-ospfunnum-puppet - Puppet, PTM and OSPF Unnumbered IPv4
cldemo-wbench-puppetserver - Puppetmaster basic configuration
cldemo-wbench-sflow-2lt22s-puppet - Setup sflow and ganglia web frontend
cldemo-wbench-sflow-2s-puppet - Setup sflow with 2s topology
cldemo-wbench-sflow-base-puppet - sflow puppet base
cldemo-wbench-vmware-demo-2lt22s-puppet - Vmware demo setup package for Puppet
cldemo-wbench-ztp-puppet - Zero touch provisioning script for Puppet


cumulus@wbench:~$ sudo apt-get install cldemo-wbench-ospfunnum-puppet

```

*  Get the switches to re-run ZTP and rerun ZTP. Got a script to do this.  Then wait about 1-2 minutes. Takes a while to execute the ZTP script that installs the puppet agent on the switches.
```
cumulus@wbench:~$ $HOME/clear_ztp_all_switches.sh
```


* Verify that the config occurred properly. Run the cldemo serverspec tests. Most tests should fail. The failed tests occur because it is checking for files found only on HW switches.
```
cumulus@wbench:~$ cd $HOME/cldemo-tests
cumulus@wbench:~/cldemo-tests$ bundle exec rake ospfunnum 2lt22s
...
.....
........
Test report summary

TARGET | TOTAL | PASSED | FAILED
-------|-------|--------|-------
leaf1  | 64    | 59     | 5
leaf2  | 64    | 59     | 5
spine1 | 58    | 53     | 5
spine2 | 58    | 53     | 5

```


## LICENSE
MIT



## TODO

* Write script to simulate ``vagrant up`` on switches after switches lose their
Vagrant managed interface.

