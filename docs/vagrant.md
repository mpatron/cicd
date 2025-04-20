# Vagrant & KVM

[index](../readme.md)

## Installation de vagrant

~~~bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
~~~

~~~bash
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-hostmanager
~~~

~~~bash
vagrant init generic/debian12 # A ne faire qu'une seule fois
vagrant box update
vagrant box prune --force
~~~

Fini le ménage, on commence par lancer la VM :

~~~bash
vagrant up --provision --provider=libvirt
vagran ssh # Connection à la VM Ubuntu 20.04 qui va porter LXC/LXD
~~~

A la fin, pour détruire la VM :

~~~bash
vagrant destroy --force
~~~

## Work arround about bug vagrant-libvirt

Work arround about bug on vagrant-libvirt ( november 2024)
[https://github.com/vagrant-libvirt/vagrant-libvirt/issues/1830#issuecomment-2447925186](https://github.com/vagrant-libvirt/vagrant-libvirt/issues/1830#issuecomment-2447925186)

~~~bash
moi@ubuntu:~/Documents/$ git clone https://github.com/vagrant-libvirt/vagrant-libvirt.git
moi@ubuntu:~/Documents/$ cd vagrant-libvirt
moi@ubuntu:~/Documents/vagrant-libvirt$ git diff
diff --git a/vagrant-libvirt.gemspec b/vagrant-libvirt.gemspec
index 457277f..13d67c7 100644
--- a/vagrant-libvirt.gemspec
+++ b/vagrant-libvirt.gemspec
@@ -21,7 +21,7 @@ Gem::Specification.new do |s|
   s.version       = VagrantPlugins::ProviderLibvirt.get_version
 
   s.add_runtime_dependency 'fog-libvirt', '>= 0.6.0'
-  s.add_runtime_dependency 'fog-core', '~> 2'
+  s.add_runtime_dependency 'fog-core', '~> 2.5.0'
   s.add_runtime_dependency 'rexml'
   s.add_runtime_dependency 'xml-simple'
   s.add_runtime_dependency 'diffy'
sudo apt  install ruby-rubygems libvirt-dev
vi vagrant-libvirt.gemspec # See up
gem build
VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install ./vagrant-libvirt-0.12.3.pre.18.gem
~~~

## Cleanning

~~~bash
for N in {0..5}; do ssh-keygen -f '/home/mickael/.ssh/known_hosts' -R 192.168.56.14${N}; done
~~~

~~~bash
sudo dnf -y install qemu-kvm libvirt virt-install
sudo systemctl enable --now libvirtd
sudo systemctl status libvirtd
systemctl enable --now virtnetworkd
sudo systemctl status virtnetworkd

# add bridge [br0]
sudo nmcli connection add type bridge autoconnect yes con-name br0 ifname br0
# set IP address for [br0]
sudo nmcli connection modify br0 ipv4.addresses 10.0.0.30/24 ipv4.method manual
# set Gateway for [br0]
sudo nmcli connection modify br0 ipv4.gateway 10.0.0.1
# set DNS for [br0]
sudo nmcli connection modify br0 ipv4.dns 10.0.0.10
# set DNS search base for [br0]
sudo nmcli connection modify br0 ipv4.dns-search jobjects.net
# remove the current interface
sudo nmcli connection del eno1
# add the removed interface again as a member of [br0]
sudo nmcli connection add type bridge-slave autoconnect yes con-name eno1 ifname eno1 master br0
# restart
sudo reboot
~~~

~~~bash
mickael@deborah:~$ sudo nmcli networking connectivity check
[sudo] Mot de passe de mickael : 
full
mickael@deborah:~$ sudo nmcli connection show 
NAME                 UUID                                  TYPE      DEVICE 
Connexion filaire 1  0a7fdf7c-ff6a-3b66-97cc-da0a43911b1d  ethernet  eno1   
lo                   03917013-9166-48ee-9075-b96935b45549  loopback  lo     
enp1s0               a2ceffd5-7ed3-4a69-8bfa-3831d8919a43  ethernet  --     
mickael@deborah:~$ sudo nmcli connection show enp1s0 
connection.id:                          enp1s0
connection.uuid:                        a2ceffd5-7ed3-4a69-8bfa-3831d8919a43
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              enp1s0
connection.autoconnect:                 oui
connection.autoconnect-priority:        0
connection.autoconnect-retries:         -1 (default)
connection.multi-connect:               0 (default)
connection.auth-retries:                -1
connection.timestamp:                   0
connection.permissions:                 --
connection.zone:                        --
connection.controller:                  br0
connection.master:                      br0
connection.slave-type:                  bridge
connection.port-type:                   bridge
connection.autoconnect-slaves:          -1 (default)
connection.autoconnect-ports:           -1 (default)
connection.down-on-poweroff:            -1 (default)
connection.secondaries:                 --
connection.gateway-ping-timeout:        0
connection.metered:                     inconnu
connection.lldp:                        default
connection.mdns:                        -1 (default)
connection.llmnr:                       -1 (default)
connection.dns-over-tls:                -1 (default)
connection.mptcp-flags:                 0x0 (default)
connection.wait-device-timeout:         -1
connection.wait-activation-delay:       -1
802-3-ethernet.port:                    --
802-3-ethernet.speed:                   0
802-3-ethernet.duplex:                  --
802-3-ethernet.auto-negotiate:          non
802-3-ethernet.mac-address:             --
802-3-ethernet.cloned-mac-address:      --
802-3-ethernet.generate-mac-address-mask:--
802-3-ethernet.mac-address-denylist:    --
802-3-ethernet.mtu:                     auto
802-3-ethernet.s390-subchannels:        --
802-3-ethernet.s390-nettype:            --
802-3-ethernet.s390-options:            --
802-3-ethernet.wake-on-lan:             default
802-3-ethernet.wake-on-lan-password:    --
802-3-ethernet.accept-all-mac-addresses:-1 (default)
bridge-port.priority:                   32
bridge-port.path-cost:                  100
bridge-port.hairpin-mode:               non
bridge-port.vlans:                      --
mickael@deborah:~$ nmcli device status
DEVICE  TYPE      STATE                  CONNECTION          
eno1    ethernet  connecté               Connexion filaire 1 
lo      loopback  connecté (en externe)  lo                  
mickael@deborah:~$
~~~


yum -y install bridge-utils
yum -y groupinstall "Virtualization Tools"
export MAIN_CONN=enp1s0
bash -x <<EOS
systemctl stop libvirtd
nmcli c delete "$MAIN_CONN"
nmcli c delete "Connexion filaire 1"
nmcli c add type bridge ifname br0 autoconnect yes con-name br0 stp off
#nmcli c modify br0 ipv4.addresses 192.168.1.18/24 ipv4.method manual
#nmcli c modify br0 ipv4.gateway 192.168.1.1
#nmcli c modify br0 ipv4.dns 192.168.1.1
nmcli c add type bridge-slave autoconnect yes con-name "$MAIN_CONN" ifname "$MAIN_CONN" master br0
systemctl restart NetworkManager
systemctl start libvirtd
systemctl enable libvirtd
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-ipforward.conf
sysctl -p /etc/sysctl.d/99-ipforward.conf
EOS

https://docs.fedoraproject.org/en-US/fedora-server/administration/virtual-routing-bridge/
https://docs.fedoraproject.org/en-US/fedora-server/administration/virtual-routing-bridge/


[[ $(sudo systemctl is-active libvirtd) ]] && sudo systemctl enable --now libvirtd
sudo virsh net-start default
