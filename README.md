# Hetzner Additional IP and Vagrant
Configure a Vagrant machine on a Hetzner Server with an addional ip.

## Step 1: Configure Server
Background Information:

```
Server IP: 10.0.10.100
Netmask: 255.255.255.192
gateway 11.10.10.12

Additional IP: 23.42.23.42
Additional Mac: 00:11:22:33:44:55
```

First we need to set the default static IP address to a bridge and remove the _eth0_ configuration:

```
auto  br0
iface br0 inet static
 address 10.0.10.100
 netmask 255.255.255.192
 gateway 11.10.10.12
 bridge_ports eth0
 bridge_stp off
 bridge_fd 1
 bridge_hello 2
 bridge_maxage 12
```
File: _/etc/network/interfaces_
(More Information about can be found here: [1])

## Step 2: Create a Vagrant File

You need to create a Vagrantfile for your project:

```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.synced_folder ".", "/vagrant", :mount_options => ["dmode=777","fmode=666"]
  config.vm.network "public_network", bridge: "br0", ip: "23.42.23.42", mac: "001122334455"
  config.vm.network "private_network", type: "dhcp"

  # change password
  config.vm.provision "shell",
    run: "always",
    inline: 'echo -e "iJ1PW0eFtK3RjzBSzu0swA\niJ1PW0eFtK3RjzBSzu0swA" | passwd vagrant'
  config.ssh.password = "iJ1PW0eFtK3RjzBSzu0swA"

 # default router
  config.vm.provision "shell",
    run: "always",
    inline: "route add default gw 10.0.10.100"

  # delete default gw on eth0
  config.vm.provision "shell",
    run: "always",
    inline: "eval `route -n | awk '{ if ($8 ==\"eth0\" && $2 != \"0.0.0.0\") print \"route del default gw \" $2; }'`"

end
```
You can finde on vagrant [2] some more information about the public networking.

As you can see, the Vagrantfile-shell-provisioner will change the default vagrant password to another one. This is important because your VM is connected to the web and everybody could access the VM with the default password.

Start the VM with ```vagrant up```. The initial startup will ask for the default password, just type _vagrant_ and it will continue. After the first boot and provisioning it will never ask again.

You should be able to ping 23.42.23.42 now and of course access it via ssh: 
```
ssh vagrant@23.42.23.42
```

That's it. Happy operating.


[1] https://wiki.hetzner.de/index.php/Netzkonfiguration_Debian#Bridged
[2] https://www.vagrantup.com/docs/networking/public_network.html

