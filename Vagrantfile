# -*- mode: ruby -*-

VM_RAM = "6144" # 1024 2048 3072 4096 6144 8192
VM_CPU = 2
# IMAGE = "fedora/41-cloud-base" # Ne fonctione pas avec fédora 41
IMAGE = "almalinux/9"
# IMAGE = "alvistack/fedora-42"
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
    libvirt.cpu_mode = "host-passthrough"
    libvirt.cpus = VM_CPU
    libvirt.disk_bus = "virtio"
    libvirt.disk_driver :cache => "writeback"
    libvirt.driver = "kvm"
    libvirt.memory = VM_RAM
    libvirt.memorybacking :access, :mode => "shared"
    # Nested virtualization allows you to run a virtual machine (VM) inside another VM
    libvirt.nested = true
    libvirt.nic_model_type = "virtio"
    # Use QEMU session instead of system connection
    libvirt.qemu_use_session = false
    libvirt.video_type = "virtio"
  end

  config.vm.define "idm" do |idm|
    idm.vm.hostname = "idm.#{DOMAIN}"
    idm.vm.network "private_network", :ip => "#{IP_BASE}.110"
  end
  config.vm.define "scm" do |scm|
    scm.vm.hostname = "scm.#{DOMAIN}"
    scm.vm.network "private_network", :ip => "#{IP_BASE}.111"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = true # default=true ou "-vvv" pour debug
    ansible.limit = "all"
    ansible.inventory_path = "provisioning/inventories/staging"
    ansible.playbook = "provisioning/provision-playbook.yml"
    ansible.extra_vars = {
      "vm_count": VM_COUNT,
      "DOMAIN": DOMAIN
    }
    ansible.groups = {
      "ipaserver" => ["idm.jobjects.net"],
      "ipaclients" => ["scm.jobjects.net"],
      "all_groups:children" => ["ipaserver", "ipaclients"],
      "all_groups:vars" => { "ansible_user" => "vagrant",
                            "ansible_password" => "vagrant",
                            "ansible_become" => true,
                            "ansible_python_interpreter" => "/usr/bin/python3",
                            "ansible_connection" => "ssh",
                            "ansible_ssh_common_args" => "-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null",
                            "ipaadmin_password" => "HelloWorld1",
                            "ipadm_password" => "HelloWorld1",
                            "ipaserver_domain" => "jobjects.net",
                            "ipaserver_realm" => "JOBJECTS.NET"
                            }, 
      "ipaserver:vars" => { "ipaserver_setup_dns" => true,
                            "ipaserver_auto_forwarders" => true},
      "ipaclients:vars" => { "ipaclient_server" => "idm.jobjects.net",
                             "ipaclient_dns_servers" => ["idm.jobjects.net"],
                             "ipaclient_configure_dns_resolver" => true},
    }
  end

end
