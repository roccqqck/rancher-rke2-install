# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_NO_PARALLEL'] = 'yes'

Vagrant.configure(2) do |config|

  config.vm.provision "shell", path: "bootstrap.sh"


  LoadbalancerCount = 2

  # Load Balancer Node
  (1..LoadbalancerCount).each do |i|
    config.vm.define "loadbalancer#{i}" do |lb|
      lb.vm.box = "ubuntu/focal64"
      lb.vm.hostname = "loadbalancer#{i}.example.com"
      lb.vm.network "private_network", ip: "192.168.56.5#{i}"
      lb.vm.provider "virtualbox" do |v|
        v.name = "loadbalancer#{i}"
        v.memory = 1024
        v.cpus = 1
      end
    end
  end

  MasterCount = 3

  # Kubernetes Master Nodes
  (1..MasterCount).each do |i|
    config.vm.define "kmaster#{i}" do |masternode|
      masternode.vm.box = "ubuntu/focal64"
      masternode.vm.hostname = "kmaster#{i}.example.com"
      masternode.vm.network "private_network", ip: "192.168.56.10#{i}"
      masternode.vm.provider "virtualbox" do |v|
        v.name = "kmaster#{i}"
        v.memory = 2048
        v.cpus = 2
      end
    end
  end

  NodeCount = 2

  # Kubernetes Worker Nodes
  (1..NodeCount).each do |i|
    config.vm.define "kworker#{i}" do |workernode|
      workernode.vm.box = "ubuntu/focal64"
      workernode.vm.hostname = "kworker#{i}.example.com"
      workernode.vm.network "private_network", ip: "192.168.56.20#{i}"
      workernode.vm.provider "virtualbox" do |v|
        v.name = "kworker#{i}"
        v.memory = 1024
        v.cpus = 1
      end
    end
  end

end
