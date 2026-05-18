# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end

  config.trigger.before :up do |trigger|
    trigger.info = "Limpando servidor DHCP do VirtualBox..."
    trigger.run = {inline: "bash -c 'VBoxManage dhcpserver remove --network HostInterfaceNetworking-vboxnet0 || true'"}
  end

  config.vm.define "arq" do |arq|
    arq.vm.hostname = "arq.kaua.pedro.devops"
    arq.vm.network "private_network", ip: "192.168.56.112"

    arq.vm.provider "virtualbox" do |vb|
      vb.name = "arq.kaua.pedro"
      vb.memory = 512
      vb.linked_clone = true

      (1..3).each do |i|
        file_to_disk = "disk-#{i}.vdi"
        unless File.exist?(file_to_disk)
          vb.customize ["createmedium", "disk", "--filename", file_to_disk, "--size", 10240]
        end
        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", file_to_disk]
      end
    end
  end

  config.vm.define "db" do |db|
    db.vm.hostname = "db.kaua.pedro.devops"
    db.vm.network "private_network", type: "dhcp"
    db.vm.provider "virtualbox" do |vb|
      vb.memory = 512
      vb.linked_clone = true
      vb.customize ["modifyvm", :id, "--macaddress1", "080027121220"]
    end
  end

  config.vm.define "app" do |app|
    app.vm.hostname = "app.kaua.pedro.devops"
    app.vm.network "private_network", type: "dhcp"

    app.vm.provider "virtualbox" do |vb|
      vb.memory = 512
      vb.linked_clone = true
      vb.customize ["modifyvm", :id, "--macaddress1", "080027121230"]
    end
  end

  config.vm.define "cli" do |cli|
    cli.vm.hostname = "cli.kaua.pedro.devops"
    cli.vm.network "private_network", type: "dhcp"

    cli.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.linked_clone = true
    end
  end
config.vm.provision "shell", inline: "mkdir -p /var/lib/apt/lists/partial && apt-get update", privileged: true
end
