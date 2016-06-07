Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.synced_folder ".", "/vagrant", :mount_options => ["dmode=777","fmode=666"]
  config.vm.network "public_network", bridge: "br0", ip: "23.42.23.42", mac: "001122334455"
  config.vm.network "private_network", type: "dhcp"

  # change password
  config.vm.provision "shell",
    run: "always",
    inline: 'echo -e "iJ1PW0eFtK3RjzBSzu0swyA\niJ1PW0eFtK3RjzBSzu0swyA" | passwd vagrant'
  config.ssh.password = "iJ1PW0eFtK3RjzBSzu0swyA"

  # default router
  config.vm.provision "shell",
    run: "always",
    inline: "route add default gw 10.0.10.100"

  # delete default gw on eth0
  config.vm.provision "shell",
    run: "always",
    inline: "eval `route -n | awk '{ if ($8 ==\"eth0\" && $2 != \"0.0.0.0\") print \"route del default gw \" $2; }'`"
end
