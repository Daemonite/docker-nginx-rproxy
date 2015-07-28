# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.require_version ">= 1.7.4"

VAGRANT_IP = ENV['RPROXY_VAGRANT_IP'] || "192.168.33.8"

# Assume Vagrant machines in this Vagrantfile use the docker provider (true for everything except dockerhost)
ENV['DEFAULT_VAGRANT_PROVIDER'] = 'docker'

VAGRANTFILE_API_VERSION = '2'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  #
  # Docker host VM
  #
  config.vm.define "dockerhost", autostart: false do |machine| 
    # Source: https://github.com/dduportal/boot2docker-vagrant-box
    machine.vm.box = 'dduportal/boot2docker'
    machine.vm.provider :virtualbox do |vmguest|
      vmguest.memory = 512
    end

    if Vagrant.has_plugin?("vagrant-vbguest")
      machine.vbguest.auto_update = false
    end

    machine.vm.synced_folder ".", "/vagrant"

    # Assign static IP to VM guest
    machine.vm.network "private_network", ip: VAGRANT_IP
    # Map Docker machine service ports from VM host to VM guest
    machine.vm.network :forwarded_port, :host => 80, :guest => 80
    machine.vm.network :forwarded_port, :host => 443, :guest => 443
    machine.vm.network :forwarded_port, :host => 8000, :guest => 8000

    # FIXME: Add a slight delay on startup to ensure the Docker daemon is running by the time we try to start any Docker machines
    machine.vm.provision :shell, :inline => "sleep 8"

  end

  #
  # Docker machines
  #

  config.vm.define "backend", autostart: true do |machine|

    machine.vm.provider :docker do |container|
      container.name = "sample_backend"
      container.image = "tutum/hello-world"
      container.env = {
        "NAME" => "Universe"
      }
      container.ports = [ "8000:80" ]
      container.vagrant_machine     = 'dockerhost'
      container.vagrant_vagrantfile = __FILE__
    end
  end

  config.vm.define "rproxy", autostart: true do |machine|

    machine.vm.provider :docker do |container|
      container.name = "nginx_rproxy"
      container.build_dir = "."
      container.volumes = [
        "/vagrant/logs/nginx:/var/log/nginx"
      ]
      container.ports = [
        "80:80",
        "443:443"
      ]
      container.link("sample_backend:backend")
      container.vagrant_machine     = 'dockerhost'
      container.vagrant_vagrantfile = __FILE__
    end
  end
  
end
