# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.require_version ">= 1.5.0"

NUM_INSTANCES = 3

BASE_IP_ADDR  = ENV['BASE_IP_ADDR'] || "192.168.65"
ETCD_DISCVERY = "#{BASE_IP_ADDR}.101"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "yungsang/coreos"

  config.vm.box_version = ">= 0.4.0"

  config.vm.define "discovery" do |discovery|
    discovery.vm.hostname = "discovery"

    discovery.vm.network :private_network, ip: ETCD_DISCVERY

    discovery.vm.network :forwarded_port, guest: 4243, host: 4243
    discovery.vm.network :forwarded_port, guest: 8080, host: 8080
    discovery.vm.network :forwarded_port, guest: 8081, host: 8081

    discovery.vm.provision :file, source: "./discovery", destination: "/tmp/vagrantfile-user-data"

    discovery.vm.provision :shell do |sh|
      sh.privileged = true
      sh.inline = <<-EOT
        mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/
      EOT
    end
  end

  (1..NUM_INSTANCES).each do |i|
    config.vm.define "core-#{i}" do |core|
      core.vm.hostname = "core-#{i}"

      core.vm.network :forwarded_port, guest: 4001, host: "400#{i}".to_i

      core.vm.network :private_network, ip: "#{BASE_IP_ADDR}.#{i+1}"

      core.vm.provision :file, source: "./user-data", destination: "/tmp/vagrantfile-user-data"

      core.vm.provision :shell do |sh|
        sh.privileged = true
        sh.inline = <<-EOT
          sed -e "s/%ETCD_DISCVERY%/#{ETCD_DISCVERY}/g" -i /tmp/vagrantfile-user-data
          mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/
        EOT
      end
    end
  end
end