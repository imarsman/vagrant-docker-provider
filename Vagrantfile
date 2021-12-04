
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'

# Require 'yaml' module
require 'yaml'

# Read details of containers to be created from YAML file
# Be sure to edit 'containers.yml' to provide container details

containers = YAML.load_file('containers.yml')

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.hostname = "vagrant-docker"
  config.vm.allow_hosts_modification = true

  containers.each do |container|
    config.vm.define container["name"] do |cntnr|
      cntnr.vm.synced_folder ".", "/vagrant", disabled: true
      # Configure the Docker provider for Vagrant
      cntnr.vm.provider "docker" do |d|
    
        # Specify port mappings, pull value from YAML file
        # If omitted, no ports are mapped!
        # d.ports = container["ports"]

        # Specify a friendly name for the Docker container, pull from YAML file
        d.name = container["name"]
    
        # Set host name
        d.create_args = ["-h", container["name"]]
    
        d.privileged = true
        d.remains_running = true
    
        # Specify the Docker image to use, pull value from YAML file
        d.build_dir = container["build_dir"]
        d.dockerfile = "Dockerfile"
    
        d.has_ssh = true   
        d.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro"]
      end
    end
  end
end
    