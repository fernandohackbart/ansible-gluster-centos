# Installation of a Gluster cluster over Centos7 


## Based on the Gluster documentation

https://wiki.centos.org/SpecialInterestGroup/Storage/gluster-Quickstart

http://blog.gluster.org/multiple-disk-mounts-with-virt-install/

Clean up the guests for reinstall
```
virsh destroy k8s-gluster1;virsh undefine k8s-gluster1;rm -rf /opt/k8s/guests/k8s-gluster1
virsh destroy k8s-gluster2;virsh undefine k8s-gluster2;rm -rf /opt/k8s/guests/k8s-gluster2
virsh destroy k8s-gluster3;virsh undefine k8s-gluster3;rm -rf /opt/k8s/guests/k8s-gluster3
```

Create the guests (requires to have the dcos-pxe configured and running as per https://github.com/fernandohackbart/ansible-pxe-centos
```
mkdir -p /opt/k8s/guests/k8s-gluster1

virt-install \
 -n k8s-gluster1 \
 --description="K8S Gluster 1 machine" \
 --os-type=Linux \
 --os-variant=generic \
 --ram=1500 \
 --vcpus=1 \
 --graphics=vnc \
 --noautoconsole \
 --disk path=/opt/k8s/guests/k8s-gluster1/k8s-gluster1.img,bus=virtio,size=7 \
 --disk path=/opt/k8s/guests/k8s-gluster1/k8s-gluster1-data1.img,bus=virtio,size=20 \
 --pxe \
 --network=bridge:dcos-br0,model=virtio,mac=52:54:00:e2:87:62

mkdir -p /opt/k8s/guests/k8s-gluster2

virt-install \
 -n k8s-gluster2 \
 --description="K8S Gluster 2 machine" \
 --os-type=Linux \
 --os-variant=generic \
 --ram=1500 \
 --vcpus=1 \
 --graphics=vnc \
 --noautoconsole \
 --disk path=/opt/k8s/guests/k8s-gluster2/k8s-gluster2.img,bus=virtio,size=7 \
 --disk path=/opt/k8s/guests/k8s-gluster2/k8s-gluster2-data1.img,bus=virtio,size=20 \
 --pxe \
 --network=bridge:dcos-br0,model=virtio,mac=52:54:00:e2:87:63

mkdir -p /opt/k8s/guests/k8s-gluster3

virt-install \
 -n k8s-gluster3 \
 --description="K8S Gluster 3 machine" \
 --os-type=Linux \
 --os-variant=generic \
 --ram=1500 \
 --vcpus=1 \
 --graphics=vnc \
 --noautoconsole \
 --disk path=/opt/k8s/guests/k8s-gluster3/k8s-gluster3.img,bus=virtio,size=7 \
 --disk path=/opt/k8s/guests/k8s-gluster3/k8s-gluster3-data1.img,bus=virtio,size=20 \
 --pxe \
 --network=bridge:dcos-br0,model=virtio,mac=52:54:00:e2:87:64

virsh start k8s-gluster1
virsh start k8s-gluster2
virsh start k8s-gluster3
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
```

Some Heketi cli commands

https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.2/html/container-native_storage_for_openshift_container_platform/chap-documentation-red_hat_gluster_storage_container_native_with_openshift_platform-heketi_cli
