# Notes From Playing with Kubernetes

I am working on migrating some infrastructure from a pure CoreOS setup with internally build systems for managing deployment, load-balancing, service-discovery, secret-deployment, and more.  This is a lot to manage with a small team, and even harder to keep on top of as things change.  Kubernetes looked like a way to take a lot of what we have put a lot of time into (Our code wrapped up in docker images) and leverage other tools that people are already using.  We really are not doing anything fancy with that part.  If it is hard, then these tools seem to be really be missing their point.

This will likely be written as a long monalog, documenting discoveries and miss-steps along the way.  I might do a slightly different write-up in the future about what I actually have discovered.

## First up play locally

I have been happy with CoreOS as a base image, and they have been building out a lot of tooling for working with Kubernetes, so it seems like a good choice to start with.  (I have also been living in the RedHat/Fedora world for a very long time, so I just know the commands better than the debian equivilants).


Ok Here we go:

First up the quckstart guide for a local deployment: https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant-single.html

Right off the batt they want me to download Vagrant from upstream. I don't really get exicted about this as it will bundle a bunch of other crap with it.  Lets at least try the Fedora packages.  For the record my development machine is using Fedora 23.

Check out this for some tips on getting Vagrant setup: https://fedoramagazine.org/running-vagrant-fedora-22/
I really try to just use libvirt whenever I can for virtualization instead of VirtualBox.  This is what I installed:
```bash
dnf install vagrant
dnf install vagrant-libvirt
```

Now back to the quickstart.  I installed kubectl as shown, although it did pain me to copy the binary into the system path, I suppose I could have just added it to my user's path, but I did not know if it might be used by another user down the road.

With `kubectl` installed I continued to spin-up the vagrant box.

```bash
 bashton   master  ~  kubernetes  coreos-kubernetes  single-node  vagrant up
WARNING: Nokogiri was built against LibXML version 2.9.2, but has dynamically loaded 2.9.3
Bringing machine 'default' up with 'libvirt' provider...
==> default: Box 'coreos-alpha' could not be found. Attempting to find and install...
    default: Box Provider: libvirt
    default: Box Version: >= 766.0.0
==> default: Loading metadata for box 'http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json'
    default: URL: http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json
The box you're attempting to add doesn't support the provider
you requested. Please find an alternate box or use an alternate
provider. Double-check your requested provider to verify you didn't
simply misspell it.

If you're adding a box from HashiCorp's Atlas, make sure the box is
released.

Name: coreos-alpha
Address: http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json
Requested provider: [:libvirt]
```

Ugh, looks likey they are using a virtualbox image.   Lets try and not do that.  There is a vagrant plugin I have used in the past vagrant-migrate to turn virtualbox images to libvirt ones.  The migrate plugin is located here `https://github.com/sciurus/vagrant-mutate` and is a painless install

```bash
vagrant plugin install vagrant-mutate
```

Looking in the `http://alpha.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json` file we see the box is located at `http://alpha.release.core-os.net/amd64-usr/891.0.0/coreos_production_vagrant.box`

```bash
 bashton   master  ~  kubernetes  coreos-kubernetes  single-node  1  vagrant mutate http://alpha.release.core-os.net/amd64-usr/891.0.0/coreos_production_vagrant.box libvirt
Downloading box coreos_production_vagrant from http://alpha.release.core-os.net/amd64-usr/891.0.0/coreos_production_vagrant.box
Extracting box file to a temporary directory.
Converting coreos_production_vagrant from virtualbox to libvirt.
    (100.00/100%)
Cleaning up temporary files.
The box coreos_production_vagrant (libvirt) is now ready to use.
```

Getting closer.  If you look at the Vagrantfile you will see they call this vagrant box coreos-alpha, but but box file we just mutated is coreos_production_vagrant. (Seems like someone is calling production alpha...)  To make the Vagrantfile happy when we run vagrant up we could edit the Vagrantfile or just move the mutated box.  I am going to move it so that anything else that refers to it does not get messed up.

```bash
bashton   master  ~  kubernetes  coreos-kubernetes  single-node  vagrant box list
coreos_production_vagrant (libvirt, 891.0.0)

bashton  ~  .vagrant.d  boxes  mv coreos_production_vagrant coreos-alpha
bashton   master  ~  kubernetes  coreos-kubernetes  single-node  vagrant box list
coreos-alpha (libvirt, 891.0.0)
```

There are still some notes about virtualbox in the Vagrantfile so we may still have to make some changes in there.
