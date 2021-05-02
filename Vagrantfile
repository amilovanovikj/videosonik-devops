Vagrant.configure("2") do |config|
  
  config.vm.box_check_update = false
  
  (11..15).each do |i|
    config.vm.define "instance#{i}" do | centos |
      centos.vm.box = "centos/7"
      centos.vm.network "private_network", ip: "192.168.32.#{i}"
    end
  end
 
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 1
  end
end
