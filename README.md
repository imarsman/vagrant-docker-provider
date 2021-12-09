# vagrant-docker-provider

I'd like to thank [rofrano](https://github.com/rofrano/vagrant-docker-provider)
for the code for this project that I was able to use to do my own
experimentation. I very much appreciate the open source ethos which I think
moves humanity forward in knowledge and collaboration.

This repo will build a docker image that can be used as a provider for
[Vagrant](https://www.vagrantup.com) as a Linux development environment.

As mentioned above, this project was forked from another which included a
multi-architecture build and storage of the built container in a remote
repository. At the moment I am just wishing to be able to have an amd64 base
Docker container that can be used to run Vagrant on a Mac M1. There is a fair
bit of the forked project's logic, which I will adjust to fit my use case up
over time.

One of my use cases is being able to use Vagrant to run an Intel Docker container
under Vagrant. I have determined that this can be done. I accomplished this by
adding the architecture as a parameter in the Dockerfile.

`FROM --platform=linux/amd64 debian:bullseye`

From Vagrant's point of view an image is created. It does not interfere with
Vagrant to be hosting an Intel container on an M1 computer. The build does take
longer for Intel than for native ARM64. You can change the Dockerfile platform
parameter to get an arm container.

`FROM --platform=linux/arm64 debian:bullseye`

## Vagrant with Ansible

I have not used Ansible much but wanted to learn how to use it locally to see
how it could be used to provision a server. There is a test in this project that
does a simple Ansible package install. I could likely also install python using
the raw module but it is easier to have python included with the Docker build
then use the apt module. Another option would be to run a playbook that
installed python using the raw module then another playbook that used the
installed python with the apt module to do the rest of the packages.

## Why Vagrant with Docker?

This was inspired by Apple's introduction of the M1 Silicon chip which is ARM
based. That means that solutions which use Vagrant and VirtualBox will not work
on Apple M1 Silicon because VirtualBox requires an Intel processors. This lead
me to find a solution for a virtual development environment that works with ARM
and thus Apple M1 computers. To do so a docker provisioner is used.

[Docker](https://www.docker.com) has introduced [Docker Desktop for Apple
silicon](https://docs.docker.com/docker-for-mac/apple-silicon/) that runs Docker
on Macs that have the Apple M1 chip. By using Docker as a provisioner for
Vagrant, we can simulate the same experience as developers using Vagrant with
VirtualBox. This is one case where you actually do want a Docker container to
behave like a VM.

## Image Contents

This image is based on Debian 11 or Ubuntu Focal 20.04 and contains  the
packages that are needed for a valid vagrant box. This includes the `vagrant`
userid with password-less `sudo` privileges. It also contains as `sshd` server.
Normally, it is considered a bad idea to run an `ssh` daemon in a Docker
container but in this case, the Docker container is emulating a Virtual Machine
(VM) to provide a development environment for vagrant to `ssh` into, so it makes
perfect sense.
;-)

## Example Vagrantfile

Here is a sample `Vagrantfile` that uses this image:

```ruby
  config.vm.define :server do |cntnr|
      cntnr.vm.synced_folder ".", "/vagrant", disabled: true
      # Configure the Docker provider for Vagrant
      cntnr.vm.provider "docker" do |d| 
        # Specify port mappings, pull value from YAML file
        # If omitted, no ports are mapped!
        # d.ports = container["ports"]

        # Specify a friendly name for the Docker container, pull from YAML file
        d.name = "test-server"
    
        # Set host name
        d.create_args = ["-h", "test-server"]
    
        d.privileged = true
        d.remains_running = true
    
        # Specify the Docker image to use, pull value from YAML file
        d.build_dir = ../../docker
        d.dockerfile = "Dockerfile"
    
        d.has_ssh = true   
        d.volumes = ["/sys/fs/cgroup:/sys/fs/cgroup:ro"]
      end
  end
```

## Task files to run routine opterations

I have included a [Taskfile](https://taskfile.dev/#/) for each test instead of a
Makefile. For the purposes of these tests a Taskfile is sufficient and for me
more pleasant to read and write. Here are the task commands for the Ansible test

```
% task
task: Available tasks for this project:
* destroy: 	    Destroy vagrant machine
* halt: 	    Halt vagrant
* run: 		    Run vagrant
* ssh: 		    ssh to vagrant
* ssh-config:   Get ssh config
* status:       Vagrant status
* up: 		    Start vagrant
```

## Command Line Usage

To use the docker provider, add the `--provider` flag to your `vagrant` command:

```sh
vagrant up --provider=docker
```

This will use this the docker image specified in your `Vagrantfile` as the base
box.

## Credits

A huge thanks to [Matthew Warman](http://warman.io) who provided the
`Dockerfile` from
[mcwarman/vagrant-provider](https://github.com/mcwarman/vagrant-docker-provider)
as the bases for my `Dockerfile` using `systemd`. He added all the magic to make
it work and I am very greateful for his generosity.
