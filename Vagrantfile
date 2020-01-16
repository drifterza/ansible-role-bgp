Vagrant.configure("2") do |config|
    config.vm.define "instance1" do |subconfig|
    subconfig.vm.network "private_network", ip: "192.168.56.101"
    subconfig.vm.box = "centos/7"
    subconfig.vm.hostname = "instance1"
    subconfig.vm.synced_folder ".", "/vagrant", owner: "vagrant",
    group: "vagrant", mount_options: ["uid=1234", "gid=1234"]
    config.vm.provision :shell, :inline  => "yum install -y epel-release"
    config.vm.provision :shell, :inline  => "yum install -y git"
    config.vm.provision "ansible" do |ansible|
        ansible.verbose = true
        ansible.galaxy_role_file = '/vagrant/tests/requirements.yml'
        ansible.galaxy_roles_path = '/vagrant/tests/roles'
        ansible.playbook = "/vagrant/tests/playbook.yml"
        ansible.inventory_path = "/vagrant/tests/inventory"
    end
end

    config.vm.define "instance2" do |subconfig|
    subconfig.vm.network "private_network", ip: "192.168.56.102"
    subconfig.vm.box = "centos/7"
    subconfig.vm.hostname = "instance2"
    subconfig.vm.synced_folder ".", "/vagrant", owner: "vagrant",
    group: "vagrant", mount_options: ["uid=1234", "gid=1234"]
    config.vm.provision :shell, :inline  => "yum install -y epel-release"
    config.vm.provision :shell, :inline  => "yum install -y git"
    config.vm.provision "ansible" do |ansible|
        ansible.verbose = true
        ansible.galaxy_role_file = '/vagrant/tests/requirements.yml'
        ansible.galaxy_roles_path = '/vagrant/tests/roles'
        ansible.playbook = "/vagrant/tests/playbook.yml"
        ansible.inventory_path = "/vagrant/tests/inventory"
    end
end
end