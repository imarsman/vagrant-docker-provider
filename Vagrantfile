# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.hostname = "debian"

  ############################################################
  # Provider for Docker on Intel or ARM (aarch64)
  ############################################################
  config.vm.provider :docker do |docker, override|
    override.vm.box = nil
    docker.image = "vagrant-provider:latest"
    docker.remains_running = true
    docker.has_ssh = true
    docker.privileged = true
    # This seems to be required but I haven't figured out why
    docker.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro"]
    # Vagrant makes a mount from current dir to /vagrant
    # Force amd64 - vagrant will figure it out with Mac M1
    # docker.create_args = ["--platform=linux/amd64"]
  end

end
