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
source ~/venv/bin/activate && pip install -r requirements.txt
source ~/venv/bin/activate && cd ~/Documents/cicd && vagrant up --provision --provider=libvirt
source ~/venv/bin/activate && cd ~/Documents/cicd/provisioning && ansible-playbook --inventory inventories/staging provision-playbook.yml
source ~/venv/bin/activate && cd ~/Documents/cicd/provisioning && ansible-playbook --inventory inventories/staging install_ipa.yml
source ~/venv/bin/activate && cd ~/Documents/cicd/provisioning && ansible-playbook --inventory inventories/staging install.yml
~~~

## Docker in almalinux 9

podman a des bug au downlowd

~~~bash
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo docker run hello-world
sudo groupadd docker
sudo usermod -aG docker $USER
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
logout
docker run hello-world
~~~

## Gitlab dans docker

Après avoir fait au moins un:

~~~bash
source ~/venv/bin/activate && cd ~/Documents/cicd/provisioning && ansible-playbook --inventory inventories/staging install_ipa.yml
~~~

Se connecter dans le vm scm :

~~~bash
vagrant ssh scm
~~~

Puis à l'interrieur faire

~~~bash
# Vérifier que les certificats pour le service HTTP de scm.jobjects.net venant du serveur FreeIPA sont là
ls -la /etc/ssl/private/http.scm.jobjects.net.pem /etc/ssl/certs/http.scm.jobjects.net.crt

# Installer podman, on est dans une almalinux/9.
sudo yum install -y podman vim
sudo rm -rf /srv/gitlab
sudo mkdir -p /srv/gitlab/{config,logs,data}
sudo chown -R $USER:$USER /srv/gitlab

# On vérifie que podman fonctionne bien (ainsi que la connectioin internet) avec
podman run quay.io/podman/hello

# Puis on télécharge l'image de gitlab-ce scm.jobjects.net
podman pull docker.io/gitlab/gitlab-ce:latest
podman run --detach --rm \
--hostname scm.jobjects.net \
--publish 8080:80 \
--publish 4430:443 \
--publish 2200:22 \
--name gitlab \
--volume /srv/gitlab/config:/etc/gitlab:Z \
--volume /srv/gitlab/logs:/var/log/gitlab:Z \
--volume /srv/gitlab/data:/var/opt/gitlab:Z \
--volume /etc/ssl/private/http.scm.jobjects.net.pem:/etc/ssl/private/http.scm.jobjects.net.pem:Z \
--volume /etc/ssl/certs/http.scm.jobjects.net.crt:/etc/ssl/certs/http.scm.jobjects.net.crt:Z \
--volume /etc/ipa/ca.crt:/usr/local/share/ca-certificates/ca.crt:Z \
docker.io/gitlab/gitlab-ce:latest
~~~

Dans vim /srv/gitlab/config/gitlab.rb (~ligne 550)
gitlab_rails['ldap_enabled'] = true
gitlab_rails['ldap_servers'] = YAML.load_file('/etc/gitlab/ldap_settings.yml')

créer le fichier /etc/gitlab/ldap_settings.yml

~~~bash
cat <<EOF >/srv/gitlab/config/ldap_settings.yml
main:
  label: 'FreeIPA'
  host: 'idm.jobjects.net'
  port: 636
  uid: 'uid'
  method: 'tls'
  bind_dn: 'uid=gitlab,cn=users,cn=accounts,dc=jobjects,dc=net'
  password: 'HelloWorld1'
  base: 'cn=users,cn=accounts,dc=jobjects,dc=net'
  user_filter: '(&(objectclass=inetUser)(|(memberOf=cn=developpeurs,cn=groups,cn=accounts,dc=jobjects,dc=net)))'
  group_base: 'cn=groups,cn=accounts,dc=jobjects,dc=net'
  admin_group: 'admins'
  block_auto_created_users: true
  verify_certificates: false
EOF
~~~

On est obligé de mettre "verify_certificates: false" car les certificats sont sur la vm et non dans le pod. Pour le mettre à true, il faut monter un volume et y mettre les certificats puis refaire le ca concatener.

ls -la /etc/ssl/private/http.scm.jobjects.net.pem /etc/ssl/certs/http.scm.jobjects.net.crt
sudo update-ca-certificates

On commence par vérifier que la connection ldap fonctionne

~~~bash
podman exec --interactive --tty gitlab gitlab-rake gitlab:ldap:check
~~~

Puis on fait le reste

~~~bash
podman exec --interactive --tty gitlab gitlab-ctl upgrade
podman exec --interactive --tty gitlab gitlab-ctl reconfigure
podman exec --interactive --tty gitlab gitlab-ctl restart
~~~

login
root
On affiche le mot de passe root

~~~bash
podman exec --interactive --tty gitlab cat /etc/gitlab/initial_root_password
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
Vrac

~~~txt
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
~~~