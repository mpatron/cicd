# Virt qemu/kvm

Ce script permet de créer des machines virtuelles avec QEMU/KVM
Il est divisé en plusieurs sections pour faciliter la compréhension
et la maintenance.

## Les images

Les images ISO sont des fichiers d'image disque qui contiennent une copie exacte d'un système de fichiers. Ils sont souvent utilisés pour distribuer des systèmes d'exploitation ou des applications. Dans ce script, nous allons créer un répertoire pour stocker les images ISO et y déplacer les fichiers ISO téléchargés.

~~~bash
mickael@deborah:~/Documents/cicd$ ls -lah ~/Téléchargements/*.iso
-rw-r--r--. 1 mickael mickael 2,0G 10 mars  18:12 /home/mickael/Téléchargements/AlmaLinux-9-latest-x86_64-minimal.iso
-rw-r--r--. 1 mickael mickael 2,8G 28 mars  11:31 /home/mickael/Téléchargements/Fedora-Silverblue-ostree-x86_64-41-1.4.iso
-rw-r--r--. 1 mickael mickael 2,3G 17 mars  15:14 /home/mickael/Téléchargements/Fedora-Workstation-Live-x86_64-41-1.4.iso
-rw-r--r--. 1 mickael mickael 5,3G 31 mars  18:18 /home/mickael/Téléchargements/ubuntu-24.10-desktop-amd64.iso
~~~

Liste des images ISO disponibles dans le répertoire de téléchargement.
Nous allons créer un répertoire pour stocker les images ISO et y déplacer les fichiers ISO téléchargés.
__osinfo-query os__

~~~bash
mickael@deborah:~/Documents/cicd$ osinfo-query os | head -n 20
 ID court             | Nom                                                | Version  | ID                                      
----------------------+----------------------------------------------------+----------+-----------------------------------------
 almalinux8           | AlmaLinux 8                                        | 8        | http://almalinux.org/almalinux/8        
 almalinux9           | AlmaLinux 9                                        | 9        | http://almalinux.org/almalinux/9        
 alpinelinux3.10      | Alpine Linux 3.10                                  | 3.10     | http://alpinelinux.org/alpinelinux/3.10 
 alpinelinux3.11      | Alpine Linux 3.11                                  | 3.11     | http://alpinelinux.org/alpinelinux/3.11 
 alpinelinux3.12      | Alpine Linux 3.12                                  | 3.12     | http://alpinelinux.org/alpinelinux/3.12 
 alpinelinux3.13      | Alpine Linux 3.13                                  | 3.13     | http://alpinelinux.org/alpinelinux/3.13 
 alpinelinux3.14      | Alpine Linux 3.14                                  | 3.14     | http://alpinelinux.org/alpinelinux/3.14 
 alpinelinux3.15      | Alpine Linux 3.15                                  | 3.15     | http://alpinelinux.org/alpinelinux/3.15 
 alpinelinux3.16      | Alpine Linux 3.16                                  | 3.16     | http://alpinelinux.org/alpinelinux/3.16 
 alpinelinux3.17      | Alpine Linux 3.17                                  | 3.17     | http://alpinelinux.org/alpinelinux/3.17 
 alpinelinux3.18      | Alpine Linux 3.18                                  | 3.18     | http://alpinelinux.org/alpinelinux/3.18 
 alpinelinux3.19      | Alpine Linux 3.19                                  | 3.19     | http://alpinelinux.org/alpinelinux/3.19 
 alpinelinux3.20      | Alpine Linux 3.20                                  | 3.20     | http://alpinelinux.org/alpinelinux/3.20 
 alpinelinux3.21      | Alpine Linux 3.21                                  | 3.21     | http://alpinelinux.org/alpinelinux/3.21 
 alpinelinux3.5       | Alpine Linux 3.5                                   | 3.5      | http://alpinelinux.org/alpinelinux/3.5  
 alpinelinux3.6       | Alpine Linux 3.6                                   | 3.6      | http://alpinelinux.org/alpinelinux/3.6  
 alpinelinux3.7       | Alpine Linux 3.7                                   | 3.7      | http://alpinelinux.org/alpinelinux/3.7  
 alpinelinux3.8       | Alpine Linux 3.8                                   | 3.8      | http://alpinelinux.org/alpinelinux/3.8
 ~~~

~~~bash
mickael@deborah:~/Documents/cicd/virt$ virt-install --osinfo list | grep -E 'fedora4|rhl9|ubuntu2'
fedora41
fedora40
fedora4
rhl9
ubuntu25.04, ubuntuplucky
ubuntu-stable-latest, ubuntu24.10, ubuntuoracular
ubuntu-lts-latest, ubuntu24.04, ubuntunoble
ubuntu23.10, ubuntumantic
ubuntu23.04, ubuntulunar
ubuntu22.10, ubuntukinetic
ubuntu22.04, ubuntujammy
ubuntu21.10, ubuntuimpish
ubuntu21.04, ubuntuhirsute
ubuntu20.10, ubuntugroovy
ubuntu20.04, ubuntufocal
~~~

## virsh

~~~
mickael@deborah:~/Documents/cicd/virt$ sudo LANG=C virsh list --all
 Id   Name   State
-----------------------
 -    idm    shut off
~~~
sudo virsh start --console idm

Supprime une VM
virsh undefine idm --remove-all-storage
Eteind une VM
virsh destroy guest1

~~~bash
mickael@deborah:~/Documents/cicd/virt$ sudo virsh net-list --all
 Nom       État    Démarrage automatique   Persistant
-------------------------------------------------------
 default   actif   oui                     oui
~~~



virt-install --name fedora-silverblue --memory 4096 --vcpus=2 --location=/home/mickael/Téléchargements/Fedora-Silverblue-ostree-x86_64-41-1.4.iso  --network bridge=nm-bridge  --graphics=vnc --console pty,target_type=serial -v