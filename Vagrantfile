BOX_IMAGE = "geerlingguy/centos7"
NODE_COUNT = 1
LS_COUNT = 1
KAFKA_COUNT = 0
ELASTIC_COUNT = 0

$script = <<-SCRIPT
date > /etc/vagrant_provisioned_at
yum install ansible -y
SCRIPT
def shared_ansible_config(ansible)
  ansible.playbook = "ansible/site.yml"
  ansible.verbose = "true"
  ansible.become = "true"
  ansible.inventory_path = "ansible/inventory/development"
end
def shared_host_options(subconfig, hostname, memory)
  subconfig.vm.network "public_network", type: "dhcp", bridge: "Intel(R) Ethernet Connection (2) I219-LM"
  subconfig.vm.box = BOX_IMAGE
  subconfig.vm.synced_folder ".", "/vagrant", type: "virtualbox"
  subconfig.vm.provision "shell", inline: $script
  subconfig.vm.hostname = hostname
  subconfig.vm.provider "virtualbox" do |vb|
    vb.memory = memory
  end
  subconfig.vm.provision "ansible_local" do |ansible|
    shared_ansible_config ansible
  end
end
Vagrant.configure("2") do |config|
  (1..NODE_COUNT).each do |j|
    config.vm.define "common#{j}" do |subconfig|
      shared_host_options subconfig, "common#{j}", "768"
    end
  end
  (1..LS_COUNT).each do |k|
    config.vm.define "logstash#{k}" do |subconfig|
      shared_host_options subconfig, "logstash#{k}", "2048"
    end
  end
  (1..ELASTIC_COUNT).each do |i|
    config.vm.define "elastic#{i}" do |subconfig|
      shared_host_options subconfig, "elastic#{i}", "4096"
      subconfig.vm.provider "virtualbox" do |vb|
         unless File.exist?("../rawdisks/elastic_disk#{i}.vdi")
            vb.customize ["createhd", "--filename", "../rawdisks/elastic_disk#{i}.vdi", "--variant", "Fixed", "--size", 20 * 1024]
         end
         vb.customize ["storageattach", :id,  "--storagectl", "IDE", "--port", 1, "--device", 0, "--type", "hdd", "--medium", "../rawdisks/elastic_disk#{i}.vdi"]
      end
    end
  end
end
