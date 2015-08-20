# Recreating Workbench like environment using libvirt-vagrant

## Requirements

* Ubuntu 14.04 on Bare metal server
* patched up libvirt from ppa for udp support
* included in this repo is the patched libvirt-vagrant gem for support of these
  features

1. Create a xml to add to libvirt of an existing bridge

### existing_bridge.xml
```
<network>
  <name>existing-bridge</name>
  <forward mode="bridge"/>
  <bridge name="br0" />
</network>

```

2. Make sure the bridge already exists

```
$ brctl show
bridge name bridge id   STP enabled interfaces
br0   8000.000000000000 no
```

3. Add the bridge definition to libvirt. Libvirt will call it `existing-bridge`
even though its really `br0`. Create the bridge definition as a persistent
bridge so that it can auto started when libvirt-bin starts

```
$ virsh net-define existing_bridge.xml
$ virsh net autostart existing-bridge

```


4. Create the VagrantFile. Requires ubuntu and Cumulus VM

Here is a sample of one device

```
```

5. install ppa version of libvirt

```
```

6. Install libvirt-vagrant with the patch for udp tunnel support


7. Then issue 'vagrant up'

8. Delete the interface created for vagrant ssh purpose.

```
virsh list

virsh edit [ ]

```
Remove first ``<interface... </interface>`` entry and save the config.

9. Restart VM using virsh

```
virsh destroy []
virsh start []
```

10. Cumulus VM should now get its eth0 IP from the wbench VM.


