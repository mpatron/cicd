# -*- mode: ruby -*-

VM_RAM = "4096" # 1024 2048 3072 4096 6144 8192
VM_CPU = 2
# IMAGE = "fedora/41-cloud-base" # Ne fonctione pas avec fédora 41
IMAGE = "almalinux/9"
# IMAGE = "generic/ubuntu2004"
# IMAGE = "generic/alma9"
DOMAIN = "jobjects.net"
IP_BASE = "192.168.56"
VM_COUNT = 1 # Nombre de VM à créer

Vagrant.configure("2") do |config|
  config.vm.box = IMAGE
  config.vm.box_check_update = false # default is true
  config.vm.boot_timeout = 600 # default (secondes) is 300

  config.vm.provider :libvirt do |libvirt|
    # Use QEMU session instead of system connection
    libvirt.qemu_use_session = false
    # Nested virtualization allows you to run a virtual machine (VM) inside another VM
    libvirt.nested = true
    libvirt.cpus = VM_CPU
    libvirt.memory = VM_RAM
  end
  config.vm.define "idm" do |idm|
    idm.vm.hostname = "idm.#{DOMAIN}"
    idm.vm.network "private_network", :ip => "#{IP_BASE}.110"
    idm.vm.provision "ansible" do |ansible|
      ansible.verbose = false # default=true ou "-vvv" pour debug
      # ansible.limit = "all"
      ansible.playbook = "provision-playbook.yml"
      ansible.extra_vars = {
        "vm_count": VM_COUNT,
        "DOMAIN": DOMAIN
      }
    end
  end
  config.vm.define "gitlab" do |gitlab|
    gitlab.vm.hostname = "gitlab.#{DOMAIN}"
    gitlab.vm.network "private_network", :ip => "#{IP_BASE}.111"
    gitlab.vm.provision "ansible" do |ansible|
      ansible.verbose = false # default=true ou "-vvv" pour debug
      # ansible.limit = "all"
      ansible.playbook = "provision-playbook.yml"
      ansible.extra_vars = {
        "vm_count": VM_COUNT,
        "DOMAIN": DOMAIN
      }
    end
  end
end
