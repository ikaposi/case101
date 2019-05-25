$script = <<-SCRIPT
echo Provisioning; installing ansible
date > /etc/vagrant_provisioned_at
yum install ansible -y
SCRIPT

Vagrant.configure("2") do |config|
# config.vm.box = "mwrock/Windows2016"
  BOX_IMAGE = "geerlingguy/centos7"
  NODE_COUNT = 1
  LS_COUNT = 0
  ELASTIC_COUNT = 0
 # config.vm.define "master" do |subconfig|
 #   subconfig.vm.box = BOX_IMAGE
 # end  
  (1..NODE_COUNT).each do |j|
    config.vm.define "common#{j}" do |subconfig|
      subconfig.vm.network "public_network", type: "dhcp", bridge: "Intel(R) Ethernet Connection (2) I219-LM"
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "common#{j}"
      subconfig.vm.synced_folder ".", "/vagrant", type: "virtualbox"
      subconfig.vm.provider "virtualbox" do |vb|
        vb.memory = "768"
      end
      subconfig.vm.provision "shell", inline: $script
      subconfig.vm.provision "ansible_local" do |ansible|
        ansible.playbook = "ansible/site.yml"
        ansible.verbose = "true"
        ansible.become = "true"
        ansible.inventory_path = "ansible/inventory/development"
      end
    end
  end
  (1..LS_COUNT).each do |k|
    config.vm.define "logstash#{k}" do |subconfig|
      subconfig.vm.network "public_network", type: "dhcp", bridge: "Intel(R) Ethernet Connection (2) I219-LM"
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "logstash#{k}"
      subconfig.vm.synced_folder ".", "/vagrant", type: "virtualbox"
      subconfig.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      subconfig.vm.provision "shell", inline: $script
    end
  end


  (1..ELASTIC_COUNT).each do |i|
    config.vm.define "elastic#{i}" do |subconfig|
      subconfig.vm.box = BOX_IMAGE
      subconfig.vm.hostname = "elastic#{i}"
      # we want a public network interface, making vm's able to resolve eachother
      subconfig.vm.network "public_network", type: "dhcp", bridge: "Intel(R) Ethernet Connection (2) I219-LM"
#      cannot disable folder sync since ansible needs to be run from within a VM (this is run on a windows computer)
#      subconfig.vm.synced_folder ".", "/vagrant", disabled: true
      # some extra storage for elasticsearch nodes
      subconfig.vm.provider "virtualbox" do |vb|
         unless File.exist?("../rawdisks/elastic_disk#{i}.vdi")
            vb.customize ["createhd", "--filename", "../rawdisks/elastic_disk#{i}.vdi", "--variant", "Fixed", "--size", 20 * 1024]
         end
         vb.memory = "4096"
         vb.customize ["storageattach", :id,  "--storagectl", "IDE", "--port", 1, "--device", 0, "--type", "hdd", "--medium", "../rawdisks/elastic_disk#{i}.vdi"]
      end
      # execution of script block in the head of this VagrantFile
      subconfig.vm.provision "shell", inline: $script
    end
  end
end
