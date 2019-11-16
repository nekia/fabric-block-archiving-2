https://docs.docker.com/install/linux/docker-ce/binaries/
https://docs.docker.com/config/daemon/#manually-create-the-systemd-unit-files
https://github.com/geerlingguy/JJG-Ansible-Windows/issues/28
https://docs.docker.com/compose/install/#alternative-install-options

## Setup OS

* Install 64bit raspbian OS on Raspberry Pi 3B
  * https://github.com/Crazyhead90/pi64/releases/tag/2018-04-17
	* Following the official install procedure to install OS
  * You need to setup wifi on Raspberry Pi via pi64-config
Building docker baseos
docker build -f config/baseos/Dockerfile \
	-t hyperledger/fabric-baseos \
	-t hyperledger/fabric-baseos:arm64-0.4.19-snapshot-83f0bd8 \

## Setup tools

* After booting up OS successfully, you need to install several tools
  * golang
  * docker
		Install Docker CE from binaries | Docker Documentation
		* https://docs.docker.com/install/linux/docker-ce/binaries/
		* You also need to setup systemd for automatic bootup of docker daemon when starting OS
			https://docs.docker.com/config/daemon/#manually-create-the-systemd-unit-files
	* docker-compose
		Install Docker Compose | Docker Documentation
		* https://docs.docker.com/compose/install/#alternative-install-options
	* Node.js
			nodesource/distributions: NodeSource Node.js Binary Distributions
			Node.js v8.x	.

## Build fabric-baseimage for ARM8 (64bit) architecture

You need to build base images (fabric-baseos, fabric-basejvm, fabric-baseimage) by yourself, because these container images
are not released officially for ARM8 (64bit) architecture.

* Clone fabric-baseimage repo on Raspberry Pi

	```
	git clone https://github.com/hyperledger/fabric-baseimage.git
	cd fabric-baseimage
	git checkout v0.4.15
	```

* Apply the following patch
  
	```
	pi@raspberrypi:~/dev/fabric-baseimage$ git diff
	diff --git a/Makefile b/Makefile
	index 868b644..93fc7f3 100644
	--- a/Makefile
	+++ b/Makefile
	@@ -38,6 +38,7 @@ DOCKER_BASE_amd64=ubuntu:xenial
	DOCKER_BASE_s390x=s390x/debian:stretch
	DOCKER_BASE_ppc64le=ppc64le/ubuntu:xenial
	DOCKER_BASE_armv7l=armv7/armhf-ubuntu
	+DOCKER_BASE_arm64=arm64v8/ubuntu:xenial
	
	DOCKER_BASE=$(DOCKER_BASE_$(ARCH))
	
	diff --git a/scripts/common/setup.sh b/scripts/common/setup.sh
	index 2b6ee5e..d65c308 100755
	--- a/scripts/common/setup.sh
	+++ b/scripts/common/setup.sh
	@@ -30,8 +30,8 @@ export GOROOT="/opt/go"
	# Install Golang
	# ----------------------------------------------------------------
	mkdir -p $GOPATH
	-ARCH=`uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|'`
	-BINTARGETS="x86_64 ppc64le s390x"
	+ARCH=`uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|'  | sed 's|aarch64|arm64|'`
	+BINTARGETS="x86_64 ppc64le s390x aarch64"
	GO_VER=1.11.5
	
	# Install Golang binary if found in BINTARGETS
	@@ -69,7 +69,7 @@ EOF
	# ----------------------------------------------------------------
	NODE_VER=8.11.3
	
	-ARCH=`uname -m | sed 's|i686|x86|' | sed 's|x86_64|x64|'`
	+ARCH=`uname -m | sed 's|i686|x86|' | sed 's|x86_64|x64|' | sed 's|aarch64|arm64|'`
	NODE_PKG=node-v$NODE_VER-linux-$ARCH.tar.gz
	SRC_PATH=/tmp/$NODE_PKG
	```

* Build
	```
	make
	```

## Build fabric-peer for ARM8 (64bit) architecture

Now you can build the block archiving enabled fabric-peer container image on your Raspberry Pi. Just follow the steps mentioned [here](README.md#setting-up-the-development-environment)

