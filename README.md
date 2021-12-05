# vagrant-docker-provider

This project was forked from another which included a multi-architecture build
and storage of the built container in a remote repository. At the moment I am
just wishing to be able to have an amd64 base Docker container that can be used
to run Vagrant on a Mac M1. There is a fair bit of the forked project's logic,
which I will adjust to fit my use case up over time.

This repo will build a docker image that can be used as a provider for
[Vagrant](https://www.vagrantup.com) as a Linux development environment.

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

## Command Line Usage

To use this provider, add the `--provider` flag to your `vagrant` command:

```sh
vagrant up --provider=docker
```

This will use this the docker image specified in your `Vagrantfile` as the base
box.

You can also run this using the provided `Makefile` with:

```sh
make run
```

## Build Multi-Archtecture Image

To build this image you must use `buildx` and build it for multiple architectures so that it can run on both Intel and ARM machines.

### Initialize builder

If you don't have a builder you must first create one:

```sh
% export DOCKER_BUILDKIT=1
% docker buildx create --use --name=qemu
qemu
% docker buildx inspect --bootstrap
```

The provided `Makefile` will do this for you with:

```sh
make init
```

This will initialize the `buildx` provider as above.

### Building the default Ubuntu image

Then you can build the multi-platform image like this (where `{account}` is the name of your Docker Hub account):

```sh
docker buildx build --file Dockerfile.ubuntu --tag {account}/vagrant-provider:ubuntu --platform=linux/amd64,linux/arm64 --push .
```

This will use QEMU to build a multi-platform image and push it to docker hub.

You can also build your image with `make`:

```sh
make build REGISTRY={your account}
```

It is better if you set an environment varaible called `REGISTRY` and it will be picked up by `make`. For example, if your Docker Hub account name is `foo` you would use:

```sh
export REGISTRY=foo
make build
```

This will ensure that all of the `make` commands will use your registry and not mine (i.e., `rofrano`)

### Building image variants

The default image is `ubuntu` but you can build the `debian` image by adding the argument `IMAGE_TAG=debian` like this:

```sh
make build IMAGE_TAG=debian
```

This will use the `Dockerfile.debian` and push to an image called `rofrano/vagrant-provider:debian`

If you want to add your own variant, just create a `Dockerfile` with the variant name as the extension and it will be picked up (e.g., `Dockerfile.alpine`). Then you can use `make build IMAGE_TAG=alpine` to build and push that image.

### Remove the buildx instance

When not needed, you can safely stop and/or remove the `buildx` containers with the following commands:

```sh
docker buildx stop
docker buildx rm
```

This can also be accomplised with:

```sh
make remove
```

## Credits

A huge thanks to [Matthew Warman](http://warman.io) who provided the `Dockerfile` from [mcwarman/vagrant-provider](https://github.com/mcwarman/vagrant-docker-provider) as the bases for my `Dockerfile` using `systemd`. He added all the magic to make it work and I am very greateful for his generosity.
