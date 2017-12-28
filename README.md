# Installation of a Gluster cluster over Centos7 


## Based on the Gluster documentation

https://wiki.centos.org/SpecialInterestGroup/Storage/gluster-Quickstart

http://blog.gluster.org/multiple-disk-mounts-with-virt-install/

Clean up the guests for reinstall
```
virsh destroy dcos-gluster1;virsh undefine dcos-gluster1;rm -rf /opt/dcos/guests/dcos-gluster1
virsh destroy dcos-gluster2;virsh undefine dcos-gluster2;rm -rf /opt/dcos/guests/dcos-gluster2
virsh destroy dcos-gluster3;virsh undefine dcos-gluster3;rm -rf /opt/dcos/guests/dcos-gluster3
```

Create the guests (requires to have the dcos-pxe configured and running as per https://github.com/fernandohackbart/ansible-pxe-centos
```
mkdir -p /opt/dcos/guests/dcos-gluster1

virt-install \
 -n dcos-gluster1 \
 --description="DCOS Gluster 1 machine" \
 --os-type=Linux \
 --os-variant=generic \
 --ram=1500 \
 --vcpus=1 \
 --graphics=vnc \
 --noautoconsole \
 --disk path=/opt/dcos/guests/dcos-gluster1/dcos-gluster1.img,bus=virtio,size=7 \
 --disk path=/opt/dcos/guests/dcos-gluster1/dcos-gluster1-data1.img,bus=virtio,size=10 \
 --pxe \
 --network=bridge:dcos-br0,model=virtio,mac=52:54:00:e2:87:62

mkdir -p /opt/dcos/guests/dcos-gluster2

virt-install \
 -n dcos-gluster2 \
 --description="DCOS Gluster 2 machine" \
 --os-type=Linux \
 --os-variant=generic \
 --ram=1500 \
 --vcpus=1 \
 --graphics=vnc \
 --noautoconsole \
 --disk path=/opt/dcos/guests/dcos-gluster2/dcos-gluster2.img,bus=virtio,size=7 \
 --disk path=/opt/dcos/guests/dcos-gluster2/dcos-gluster2-data1.img,bus=virtio,size=10 \
 --pxe \
 --network=bridge:dcos-br0,model=virtio,mac=52:54:00:e2:87:63

mkdir -p /opt/dcos/guests/dcos-gluster3

virt-install \
 -n dcos-gluster3 \
 --description="DCOS Gluster 3 machine" \
 --os-type=Linux \
 --os-variant=generic \
 --ram=1500 \
 --vcpus=1 \
 --graphics=vnc \
 --noautoconsole \
 --disk path=/opt/dcos/guests/dcos-gluster3/dcos-gluster3.img,bus=virtio,size=7 \
 --disk path=/opt/dcos/guests/dcos-gluster3/dcos-gluster3-data1.img,bus=virtio,size=10 \
 --pxe \
 --network=bridge:dcos-br0,model=virtio,mac=52:54:00:e2:87:64

virsh start dcos-gluster1
virsh start dcos-gluster2
virsh start dcos-gluster3
```

Clean the known_hosts if reinstalling
```
ssh-keygen -R 192.168.40.44 -f /home/fernando.hackbart/.ssh/known_hosts
ssh-keygen -R 192.168.40.45 -f /home/fernando.hackbart/.ssh/known_hosts
ssh-keygen -R 192.168.40.46 -f /home/fernando.hackbart/.ssh/known_hosts
```

Test the ssh
```
ssh root@192.168.40.44 date
ssh root@192.168.40.45 date
ssh root@192.168.40.46 date
```

```
ansible-playbook -i hosts gluster.yml
ansible-playbook -i hosts heketi.yml
```

```
export HEKETI_CLI_SERVER=http://localhost:8080
heketi-cli cluster list
heketi-cli volume create --size=1
```