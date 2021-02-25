---
layout: default
title: docker
---

{: .title}
Docker - Launching X apps

* TOC
{: toc}

# Dockerfile
Let's say we also want a `share` volume.

## Non-opengl applications

	from debian:stable

	arg user=stephanie
	arg wd=/home/$arg/share/

	workdir $wd
	user root
	run apt update
	run apt dist-upgrade
	run apt install x11-apps

	run groupadd -r $user && useradd -r -m -s /bin/bash -g $user $user
	run chown -R $user:$user $wd

	user $user

## Opengl applications

	from debian:stable

	arg user=stephanie
	arg wd=/home/$arg/share/

	workdir $wd
	user root
	run apt update
	run apt dist-upgrade

	RUN set -xe; \
		apt update; \
		apt install -y xvfb llvm-7-dev wget g++ pkg-config zlib1g-dev x11proto-gl-dev libxext-dev libxcb-dri2-0-dev libx11-xcb-dev libxcb-xfixes0-dev libdrm-dev libexpat1-dev make python-dev; \
		wget "https://mesa.freedesktop.org/archive/mesa-19.0.1.tar.xz"; \
		tar Jxf mesa-19.0.1.tar.xz; \
		rm -f mesa-19.0.1.tar.xz; \
		cd mesa-19.0.1; \
		./configure --enable-autotools --enable-llvm --with-llvm-prefix=/usr/lib/llvm-7 --enable-glx=gallium-xlib --with-gallium-drivers=swrast,swr --disable-dri --disable-gbm --disable-egl --enable-gallium-osmesa --prefix=/usr/local; \
		make; \
		make install; \
		cd .. ; \
		rm -rf mesa-19.0.1;

	run apt install pymol
	run groupadd -r $user && useradd -r -m -s /bin/bash -g $user $user
	run chown -R $user:$user $wd

	user $user

# Build and create container
## Linux
Move to the shared directory on the host and copy the `Dockerfile` there.

	docker build . --tag xapps
	docker run -it -e DISPLAY=$DISPLAY --net=host -v /host/path:/home/stephanie/share xapps
	# Or!
	docker run -it -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix/ -v /host/path:/home/stephanie/share xapps

## Windows

	$hostip = (Get-NetIPAddress -AddressState Preferred -AddressFamily IPv4 -InterfaceAlias Wi-Fi | Select-Object IPAddress).IPAddress
	docker run -it --entrypoint pymol --rm -v C:\Users\AstridMusnier\Documents\MAbSilico\Projets\Pymol_data:/home/stephanie/share -e DISPLAY=${hostip}:0.0 -e XAUTHORITY xapps

