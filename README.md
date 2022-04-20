# vm2docker
# Conversão de uma aplicação em máquina virtual para um contêiner em Docker

1. O processo inicia-se com a atualização do sistema operacional e a instalação dos pacotes necessários para utilização do qemu. Segue:

```console 
sudo update && apt install qemu-utils
```

2. Uma vez instalados os pacotes é necessário habilitar o módulo do protocolo nbd. Segue:

```console
sudo modprobe nbd
```

3. Pode-se verificar que os módulos foram habilitados com o comando abaixo. Segue:

```console
sudo ls /dev/nbd*
```

:computer: Espera-se uma saída desta forma:
```shell-session
sudo ls /dev/nbd*
/dev/nbd0    /dev/nbd1   /dev/nbd13  /dev/nbd3  /dev/nbd7
/dev/nbd0p1  /dev/nbd10  /dev/nbd14  /dev/nbd4  /dev/nbd8
/dev/nbd0p2  /dev/nbd11  /dev/nbd15  /dev/nbd5  /dev/nbd9
/dev/nbd0p3  /dev/nbd12  /dev/nbd2   /dev/nbd6
```

4. Após isso, é necessário conectar a imagem em .vmdk em um dispositivo virtual nbd. Segue:
```console
sudo qemu-nbd -c /dev/nbd0 -r image.vmdk
```

5. Uma vez mapeado é necessário montar a partição raiz deste dispositivo(nbd0). Neste caso, a mesma é identificada como nbd0p5 e será montada no diretório /mnt.
```console
sudo mount -o -ro,noload /dev/nbd0p5 /mnt
```
6. Uma vez montado no diretório /mnt é necessário empacotar e comprimir este diretório. Para tanto pode-se utilizar a ferramenta tar. Segue:

```console
sudo tar -C /mnt -czf image.tar.gz .
```
7. Em posse do arquivo. tar.gz é necessário importá-lo para docker a fim de transformá-lo em uma imagem, ou seja, o molde para o contêiner. Segue:

```console
sudo docker import image.tar.gz image:1.0
```
Se tudo ocorreu como esperado pode-se criar um contêiner de teste a partir da imagem. Segue:

```console
sudo docker run --rm -it --name image image:1.0 /bin/bash
```
:computer: E tendo como resultado o shell do contêiner:
```console
root@9d78ff335b6f:/# 
```
