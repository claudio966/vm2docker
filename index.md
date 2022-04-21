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

This output is expected:
```shell-session
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
foo@bar:~$ sudo mount -o -ro,noload /dev/nbd0p5 /mnt
```
6. Once mounted at /mnt directory it's necessary pack and compress this directory in a tar.gz file. For that, you can use the tar program.

```console
foo@bar:~$ sudo tar -C /mnt -czf image.tar.gz .
```
7. In possess of this file, is necessary import to docker with propose to turn it in an image. That is the shape for the future container.

```console
foo@bar:~$ sudo docker import image.tar.gz image:1.0

```
If everything went as expected, you can build the container to test from the previous image:

```console
foo@bar:~$ sudo docker run --rm -it --name image image:1.0 /bin/bash
```
:computer: And with this result. The shell of your application !
```console
root@9d78ff335b6f:/# 
```
## Convert a VM that Use Partion with LVM Implementation
1. For partiton created with LVM, it's necessary an intermediate procedure between third and the fourth previous steps, that is, convert the vmdk format image to raw format image. For this, it's used the utilitarian qemu-img:
```console
foo@bar:~$ sudo qemu-img convert -f vmdk image.vmdk -O raw image.raw
```

2. In possess the raw image, you can change a bit the fourth step. With this approach:
```console
foo@bar:~$ sudo qemu-nbd -c /dev/ndb0 -r image.raw
```
