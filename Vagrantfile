# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "puppetlabs/centos-7.2-64-nocm"
  #config.vm.box = "centos/7"
  
  # Stepup-Deploy Ansible playbook takes the hostname from the inventory which is generated by Vagrant
  # We define the vm here with it's full hostname because that is what is put in the generated inventory
  # Tip: After spinning up de VM(s) with vagrant you can use bash 4 commandline completion for completing hostname in Vagrant
  #      With ansible-playbook you can use e.g. "-l app*" to safe typing
  config.vm.define "app.stepup.example.com" do |app|
    #app.vm.hostname = "app.stepup.example.com"
    # Defining a private network through vagrant proves to be too unreliable
    # Create a 192.168.66.0/24 network manually and add a nic to the VM in vmware/virtualbox manually
    #app.vm.network "private_network", ip: "192.168.66.3", :netmask => "255.255.255.0"
    app.vm.provider "vmware_fusion" do |v|
      v.vmx["memsize"] = "1536"
      v.vmx["numvcpus"] = "1"
      # puppetlabs centos box uses ens33 nic
      v.vmx["ethernet0.pciSlotNumber"] = "33"
      # Add second nic for vmnet2 (configure this manually in vmware as host based, 192.168.66.0/24)
      v.vmx["ethernet1.present"] = "TRUE"
      v.vmx["ethernet1.connectionType"] = "custom"
      v.vmx["ethernet1.virtualDev"] = "e1000"
      v.vmx["ethernet1.wakeOnPcktRcv"] = "FALSE"
      v.vmx["ethernet1.addressType"] = "generated"
      v.vmx["ethernet1.vnet"] = "vmnet2"
      v.vmx["ethernet1.pciSlotNumber"] = "32"
  #v.gui = true
    end
    app.vm.provider "virtualbox" do |v|
      v.memory = 1536
    end
  end

  config.vm.define "manage.stepup.example.com" do |manage|
    #manage.vm.hostname = "manage.stepup.example.com"
    # manage.vm.network "private_network", ip: "192.168.66.4", :netmask => "255.255.255.0"
    manage.vm.provider "vmware_fusion" do |v|
      v.vmx["memsize"] = "1536"
      v.vmx["numvcpus"] = "1"
      # puppetlabs centos box uses ens33 nic
      v.vmx["ethernet0.pciSlotNumber"] = "33"
      # Add second nic for vmnet2 (configure this manually in vmware as host based, 192.168.66.0/24)
      v.vmx["ethernet1.present"] = "TRUE"
      v.vmx["ethernet1.connectionType"] = "custom"
      v.vmx["ethernet1.virtualDev"] = "e1000"
      v.vmx["ethernet1.wakeOnPcktRcv"] = "FALSE"
      v.vmx["ethernet1.addressType"] = "generated"
      v.vmx["ethernet1.vnet"] = "vmnet2"
      v.vmx["ethernet1.pciSlotNumber"] = "32"
      #v.gui = true
    end
    manage.vm.provider "virtualbox" do |v|
      v.memory = 1536
    end
  end


  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision.yml"
    ansible.groups = {
      "app" => ["app.stepup.example.com"],
      "stepup-app" => ["app.stepup.example.com"],
      #"dbcluster" => ["db.stepup.example.com"],
      "dbcluster:children" => ["stepup-app"],
      "manage" => ["manage.stepup.example.com"],
      "es:children" => ["manage"],
      "proxy:children" => ["stepup-app"],
      "stepup-gateway:children" => ["stepup-app"],
      "stepup-selfservice:children" => ["stepup-app"],
      "stepup-ra:children" => ["stepup-app"],
      "stepup-middleware:children" => ["stepup-app"],
      "stepup-tiqr:children" => ["stepup-app"],
      "stepup-keyserver:children" => ["stepup-app"],
      "lb" => [], # Don't use a sparate lb, use the proxy role insstead. It set's up a simple reverse proxy using nginx op the app server
      "ks" => [],
      "ks:children" => ["app"]
    }
    ansible.host_vars = {
          "app.stepup.example.com" => {"host_ipv4" => "192.168.66.3", "backend_ipv4" => "192.168.66.3"},
          "manage.stepup.example.com" => {"host_ipv4" => "192.168.66.4"}
          #"db.stepup.example.com" => {"host_ipv4" => "192.168.66.5", "backend_ipv4" => "192.168.66.5" }
        }
    #ansible.verbose = "vvvv"
  end

end
