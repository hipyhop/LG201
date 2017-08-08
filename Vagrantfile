Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.define "ws-1" do |instance|
    instance.vm.network "private_network", ip: "192.168.32.21"
  end

  config.vm.define "ws-2" do |instance|
    instance.vm.network "private_network", ip: "192.168.32.22"
  end


  config.vm.define "lb-1" do |instance|
    instance.vm.network "private_network", ip: "192.168.32.10"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.groups = {
      "webservers" => [ "ws-1", "ws-2" ],
      "loadbalancers" => [ "lb-1" ]
    }
    ansible.playbook = "common.yml"
    ansible.limit = "all"
    #ansible.verbose = "vvv"
  end

end
