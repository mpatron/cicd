# readme (on working 🚧 🏗️ ...)

[https://docs.redhat.com/en/documentation/red_hat_ceph_storage/4/html/installation_guide/red-hat-ceph-storage-considerations-and-recommendations#tuning-considerations-for-the-linux-kernel-when-running-ceph_install](https://docs.redhat.com/en/documentation/red_hat_ceph_storage/4/html/installation_guide/red-hat-ceph-storage-considerations-and-recommendations#tuning-considerations-for-the-linux-kernel-when-running-ceph_install)

![Cepth architecture](images/cepth_basic_cluster.svg "Cepth architecture")

[vagrant.md](docs/vagrant.md)

~~~bash
[mpatron@node0 ~]$ python3 -m venv ~/venv
[mpatron@node0 ~]$ source ~/venv/bin/activate
(venv) [mpatron@node0 ~]$ python3 -m pip install --upgrade pip
(venv) [mpatron@node0 ~]$ python3 -m pip install ansible
(venv) [mpatron@node0 ~]$ ansible --version
ansible [core 2.15.12]
  config file = None
  configured module search path = ['/home/mpatron/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/mpatron/venv/lib64/python3.9/site-packages/ansible
  ansible collection location = /home/mpatron/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/mpatron/venv/bin/ansible
  python version = 3.9.18 (main, Jul  3 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/home/mpatron/venv/bin/python3)
  jinja version = 3.1.4
  libyaml = True
(venv) [mpatron@node0 ~]$ deactivate
[mpatron@node0 ~]$
~~~

Dépendances :
Alors en fait rien sur galaxy, mais pour pip, il y en a une sur importante, c'est passlib pour les mots de passe. Ne pas oublier de faire "pip install -r requirements.txt".

~~~bash
source ~/venv/bin/activate
# ansible-galaxy collection install -r requirements.yml --ignore-certs
# ansible-galaxy role install -r requirements.yml  --ignore-certs
# En une commande :
ansible-galaxy install --force --ignore-certs --role-file requirements.yml
pip install -r requirements.txt
vagrant up --provision --provider=libvirt
cd provisioning && ansible-playbook --inventory inventories/staging provision-playbook.yml
cd provisioning && ansible-playbook --inventory inventories/staging install-cicd.yml
~~~

## Ceph install

~~~bash
git clone https://github.com/ceph/ceph-ansible.git
cd ceph-ansible/
git tag
git checkout stable-9.0
~~~

Générer le fichier de configuration avec toutes les valeurs par default :

~~~bash
ansible-config init --disabled -t all > ansible-all-defaults.cfg
~~~


https://developer.hashicorp.com/vagrant/tutorials/networking-provisioning-operations/getting-started-networking
https://wiki.libvirt.org/VirtualNetworking.html
https://vagrant-libvirt.github.io/vagrant-libvirt/configuration.html#networks
https://www.server-world.info/en/note?os=Fedora_41&p=kvm&f=1

sudo dnf install libvirt vagrant vagrant-libvirt vagrant-sshfs
vagrant destroy -f && vagrant up && vagrant reload && vagrant ssh idm --command /bin/sh -c 'sudo ip a'
sudo virsh net-edit vagrant-libvirt
sudo virsh net-list
virsh -c qemu:///system net-list
dnf list vagrant-libvirt
vagrant plugin list
vagrant plugin install vagrant-libvirt
vagrant plugin update vagrant-libvirt
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --info-zone=libvirt
sudo virsh net-start --network vagrant-libvirt
mickael@deborah:~/Documents/douanes/cicd-ml$ sudo virsh net-list --all
 Nom               État    Démarrage automatique   Persistant
---------------------------------------------------------------
 default           actif   oui                     oui
 mk-minikube       actif   oui                     oui
 vagrant-libvirt   actif   non                     oui
mickael@deborah:~/Documents/douanes/cicd-ml$ nmcli connection show -a
NAME                 UUID                                  TYPE      DEVICE 
Connexion filaire 1  0a7fdf7c-ff6a-3b66-97cc-da0a43911b1d  ethernet  eno1   
lo                   6a5891af-d29e-49b0-b2b6-d1124b40bef5  loopback  lo     
virbr0               766b9e75-abcc-4ea6-af91-e407a370789b  bridge    virbr0 
virbr2               79e22592-d6df-44f4-a986-812d80b98ef8  bridge    virbr2
sudo ls -lah /var/lib/libvirt/qemu
