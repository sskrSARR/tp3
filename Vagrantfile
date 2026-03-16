Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  config.vm.define "server-dba" do |dba|
    dba.vm.hostname = "server-dba"
    dba.vm.network "private_network", ip: "192.168.56.18"
  end

  config.vm.define "server-back" do |back|
    back.vm.hostname = "server-back"
    back.vm.network "private_network", ip: "192.168.56.19"
    back.vm.network "forwarded_port", guest: 8080, host: 8080
  end

  config.vm.define "server-front" do |front|
    front.vm.hostname = "server-front"
    front.vm.network "private_network", ip: "192.168.56.20"
    front.vm.network "forwarded_port", guest: 80, host: 8081

    front.vm.provider "virtualbox" do |vb|
    vb.memory = 4096
    vb.cpus = 2
    end
  end

end