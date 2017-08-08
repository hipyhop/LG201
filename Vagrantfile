Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  # How many webserver should be launched
  N_WEBSERVERS = 2

  (1..N_WEBSERVERS).each do |machine_id|

    config.vm.define "ws-#{machine_id}" do |instance|
      instance.vm.network "private_network", ip: "192.168.32.#{20+machine_id}"
      instance.vm.hostname = "web-#{machine_id}"
    end

  end

  # config.vm.define "lb-1" do |instance|
  #   instance.vm.network "private_network", ip: "192.168.32.10"
  # end

  config.vm.provision "ansible" do |ansible|
    ansible.groups = {
      "webservers" => (1..N_WEBSERVERS).map {|n| "ws-" + n.to_s},
      "loadbalancers" => [ "lb-1" ]
    }
    ansible.playbook = "common.yml"
    ansible.limit = "all"
    #ansible.verbose = "vvv"
  end

end
