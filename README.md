# vagrant-centos-7-cloud-init-nocloud-iso

Demonstration of Cloud-Init with a CentOS-7 Vargrant Box, the VirtualBox provider and a NoCloud ISO datasource as an alternative to using a [mock metadata service](https://gist.github.com/jdeathe/e4e7943e48b23f48e16460630ac004a6).

Cloud-Init is used to install `docker-latest` onto the [jdeathe/centos-7-x86_64-minimal-cloud-init-en_us](https://atlas.hashicorp.com/jdeathe/boxes/centos-7-x86_64-minimal-cloud-init-en_us) base box.

## Usage

### Prerequisites (with versions tested)

Host operating system: OSX El Capitan

- [VirtualBox](https://www.virtualbox.org) (5.1.20)
- [Vagrant](https://www.vagrantup.com) (1.9.3)

### Populate user-data

Change the contents of `iso/data/user-data` as necessary, using the [Cloud-Init Documentation](https://cloudinit.readthedocs.io/en/latest/topics/examples.html) for reference.

### Start a VirtualBox VM

```
$ vagrant up
```

### Vagrant Usage

For additional usage - refer to the vagrant help.

```
$ vagrant -h
```
