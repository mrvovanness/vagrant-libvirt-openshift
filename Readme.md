### Vagrant config to play with openshift, uses libvirt provider. 
One master and one node currently

Before `vagrant up` update libvirt network `vagrant-libvirt`
```
sudo virsh net-edit vagrant-libvirt

```

Add this line under the `mac` field

```
<domain name='openshift.local' localOnly='yes'/>
```

then restart network

```
sudo virsh net-destroy vagrant-libvirt
sudo virsh net-start vagrant-libvirt
```

