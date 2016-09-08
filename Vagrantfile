Vagrant.configure(2) do |config|
  config.vm.define "kn8s" do |kn8s|
    kn8s.vm.box = "ubuntu/trusty64"
    kn8s.vm.network "private_network", ip: "192.168.0.250"
    kn8s.vm.hostname = "kn8s.demo.com"
    kn8s.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end 
  end
end
