# vm2docker
# Conversion of Virtual Machine Application for a Docker Conteiner

## The standart Procedure
1. The process initialize with the update of your operational system and, after that, the installation of the necessary packages for utilization of qemu program. Following.
```console 
foo@bar:~$ sudo update && apt install qemu-utils
```

2. Once installed, it's necessary to enable the nbd protocol modules:
```console
foo@bar:~$ sudo modprobe nbd
```

3. You can verify that modules was enabled with below command:
```console
foo@bar:~$ sudo ls /dev/nbd*
```

- This output is expected:
```console
foo@bar:~$ sudo ls /dev/nbd*
/dev/nbd0    /dev/nbd1   /dev/nbd13  /dev/nbd3  /dev/nbd7
/dev/nbd0p1  /dev/nbd10  /dev/nbd14  /dev/nbd4  /dev/nbd8
/dev/nbd0p2  /dev/nbd11  /dev/nbd15  /dev/nbd5  /dev/nbd9
/dev/nbd0p3  /dev/nbd12  /dev/nbd2   /dev/nbd6
```

4. After that, it's necessary connect the .vmdk image in a nbd virutal device:
```console
foo@bar:~$ sudo qemu-nbd -c /dev/nbd0 -r image.vmdk
```

5. Once mapped, it's necessary to mount the root partition of this device(ndb0). In this case, oneself is identified as nbd0p5 and will be mounted at /mnt directory.
```console
foo@bar:~$ sudo mount -ro,noload /dev/nbd0p5 /mnt
```
6. Once mounted at /mnt directory it's necessary pack and compress this directory in a tar.gz file. For that, you can use the tar program.
```console
foo@bar:~$ sudo tar -C /mnt -czf image.tar.gz .
```
7. In possess of this file, is necessary import to docker with propose to turn it in an image. That is the shape for the future container.
```console
foo@bar:~$ sudo docker import image.tar.gz image:1.0
```
- If everything went as expected, you can build the container to test from the previous image:
```console
foo@bar:~$ sudo docker run --rm -it --name image image:1.0 /bin/bash
```
- And with this result. The shell of your application !
```console
root@9d78ff335b6f:/# 
```

## Convert a VM that Use Partition with LVM Implementation
1. Now assume that you have this configuration for device /dev/nbd0:
```console
foo@bar:~$ sudo fdisk -l /dev/nbd0
Device Boot Start     End   Sectors Size Id Type
/dev/nbd1p1 *             2048    499711    497664    243M 83 Linux
/dev/nbd1p2             501758 251656191 251154434  119,8G  5 Estendida
/dev/nbd1p5             501760 251656191 251154432  119,8G 8e Linux LVM
```
It's clearly there are a different type of partitions, that is, LVM partitions.

2. For this partitions that use LVM implementation, it's necessary additional step to seek the right device for mount. In this case, it's used the `lvs` command. This, command seek all logical device of LVM partition in your machine, as the below example:
```console
foo@bar:~$ sudo lvs 
  LV     VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home   csic04-vg -wi-a-----   9,31g                                                    
  root   csic04-vg -wi-a-----   9,31g                                                    
  swap_1 csic04-vg -wi-a-----  <2,00g                                                    
  tmp    csic04-vg -wi-a-----  <1,37g                                                    
  var    csic04-vg -wi-a----- <97,77g  
```
In this case, the lvs command find out 5 logical partitions

If is necessary to mount the root partition
