### Gavin's notes on getting gitian builds up and running using KVM:###

These instructions distilled from:
[  https://help.ubuntu.com/community/KVM/Installation](  https://help.ubuntu.com/community/KVM/Installation)
... see there for complete details.

You need the right hardware: you need a 64-bit-capable CPU with hardware virtualization support (Intel VT-x or AMD-V). Not all modern CPUs support hardware virtualization.

You probably need to enable hardware virtualization in your machine's BIOS.

You need to be running a recent version of 64-bit-Ubuntu, and you need to install several prerequisites:

	sudo apt-get install ruby apache2 git apt-cacher-ng python-vm-builder qemu-kvm

Sanity checks:

	sudo service apt-cacher-ng status  # Should return apt-cacher-ng is running
	ls -l /dev/kvm   # Should show a /dev/kvm device

Check that you have the 'kvm' group, or you won't have permissions to use kvm:
  groups    # should include 'kvm' in the list
If the group is missing:
  sudo usermod -aG kvm <your_username>    # will need to log out and back in to take effect

Once you've got the right hardware and software:

	git clone git://github.com/(user)/slimcoin.git
	git clone git://github.com/devrandom/gitian-builder.git
	mkdir gitian-builder/inputs

	# Create base images
	cd gitian-builder
	bin/make-base-vm --suite precise --arch i386
	bin/make-base-vm --suite precise --arch amd64  # needed for SLM?
	cd ..

	# Get inputs (see doc/release-process.md for exact inputs needed and where to get them)
	...

	# For further build instructions see doc/release-notes.md
	...

	# To build
	cd slimcoin
	git pull
	cd ../gitian-builder
	git pull
	./bin/gbuild --commit slimcoin=HEAD ../slimcoin/contrib/gitian-descriptors/gitian.yml

---------------------

`gitian-builder` now also supports building using LXC. See
[  https://help.ubuntu.com/12.04/serverguide/lxc.html](  https://help.ubuntu.com/12.04/serverguide/lxc.html)
... for how to get LXC up and running under Ubuntu.

If your main machine is a 64-bit Mac or PC with a few gigabytes of memory
and at least 10 gigabytes of free disk space, you can `gitian-build` using
LXC running inside a virtual machine.

Here's a description of Gavin's setup on OSX 10.6

1: Download and install VirtualBox from [https://www.virtualbox.org/](https://www.virtualbox.org/)

2: Download the 64-bit Ubuntu Desktop 12.04 LTS .iso CD image from
   [http://www.ubuntu.com/](http://www.ubuntu.com/)

3: Run VirtualBox and create a new virtual machine, using the Ubuntu .iso (see the [VirtualBox documentation](https://www.virtualbox.org/wiki/Documentation) for details). Create it with at least 2 gigabytes of memory and a disk that is at least 20 gigabytes big.

4: Inside the running Ubuntu desktop, install:
	
	sudo apt-get install debootstrap lxc ruby apache2 git apt-cacher-ng python-vm-builder

4.5: If you are running Ubuntu 14.04 (Trusty) as your VM, you will need to set the following (you may want to add it to your `.profile`):
	
	export LXC_EXECUTE=lxc-execute
	
4.6: You'll probably also need to set these:

	export LXC_BRIDGE=lxcbr0
	export GITIAN_HOST_IP=`ifconfig $LXC_BRIDGE|grep 'inet addr:'|sed 's/^.*inet addr:\([^ ]*\) .*$/\1/'`
	export LXC_GUEST_IP=`echo $GITIAN_HOST_IP|sed 's/^\(\([^.]*\.\)*\).*$/\15/'`

4.7: And add the following to `/etc/sudoers.d/gitian-lxc` to avoid having to type your password for every build:

	%sudo ALL=NOPASSWD: /usr/bin/lxc-start
	%sudo ALL=NOPASSWD: /usr/bin/lxc-execute

5: Still inside Ubuntu, tell gitian-builder to use LXC, then follow the "Once you've got the right hardware and software" instructions above:

	export USE_LXC=1
	git clone git://github.com/bitcoin/bitcoin.git
	... etc

