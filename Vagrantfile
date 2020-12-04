# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/bionic64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./vagrant_data", "/data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.define "dory"
  config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     # vb.gui = true
     
 
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
     vb.customize ["modifyvm", :id, "--usb", "on"]
     vb.customize ["usbfilter", "add", "0",
                   "--target", :id,
                   "--name", "FTDI Dual RS232",
                   "--product", "Dual RS232",
                   "--vendorid", "0x0403",
                   "--productid", "0x6010"]
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL

     echo "INSTALLING SYSTEM DEPENDENCIES" 1>&2

     apt-get update
     apt-get install -y build-essential git libftdi-dev libftdi1 doxygen python3-pip libsdl2-dev curl cmake libusb-1.0-0-dev scons libsndfile1-dev rsync autoconf automake texinfo libtool pkg-config libsdl2-ttf-dev
     curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
     apt-get install git-lfs
     git lfs install

     echo "SETTING USB PERMISSIONS" 1>&2

     usermod -a -G dialout root
     touch 90-ftdi_gapuino.rules
     echo 'ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6010", MODE="0666", GROUP="dialout"'> 90-ftdi_gapuino.rules
     echo 'ATTRS{idVendor}=="15ba", ATTRS{idProduct}=="002b", MODE="0666", GROUP="dialout"'>> 90-ftdi_gapuino.rules
     mv 90-ftdi_gapuino.rules /etc/udev/rules.d/
     udevadm control --reload-rules && sudo udevadm trigger

     cd /data

     echo "INSTALLING GAP SDK" 1>&2
     
     git clone https://github.com/GreenWaves-Technologies/gap_riscv_toolchain_ubuntu_18.git
     cd gap_riscv_toolchain_ubuntu_18
     mkdir -p /usr/lib/gap_riscv_toolchain
     cp -rf ./* /usr/lib/gap_riscv_toolchain
     cd ..

     git clone https://github.com/GreenWaves-Technologies/gap_sdk.git
     cd gap_sdk
     git submodule update --init --recursive
     git checkout c6494b97314470446674bb468d31e4391fb187e9 
     pip3 install -r requirements.txt
     source configs/gapuino_v2.sh
     make all
     cd ..

     echo "INSTALLING DORY" 1>&2

     cd /data
     git clone https://github.com/pulp-platform/dory
     cd dory
     git submodule update --init --recursive
     pip3 install mako==1.0.12 numpy==1.18.4 pandas==1.0.3 onnx==1.5.0 ortools==7.5.7466

     echo "INSTALLATION COMPLETE" 1>&2
   SHELL
end
