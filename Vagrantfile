Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  # How many webserver should be launched
  N_DOCKERHOSTS = 2

  (1..N_DOCKERHOSTS).each do |machine_id|

    config.vm.define "docker-#{machine_id}" do |instance|
      instance.vm.network "private_network", ip: "192.168.32.#{20+machine_id}"
      instance.vm.hostname = "docker-#{machine_id}"
    end

  end

  config.vm.define "lb-1" do |instance|
    instance.vm.network "private_network", ip: "192.168.32.10"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.groups = {
      "docker-hosts" => (1..N_DOCKERHOSTS).map {|n| "docker-" + n.to_s},
      "docker-hosts:vars" => {
          "http_port" => 80
      },
      "loadbalancers" => [ "lb-1" ]
    }
    ansible.playbook = "common.yml"
    ansible.limit = "all"
    #ansible.verbose = "vvv"
  end

end
