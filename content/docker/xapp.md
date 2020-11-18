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

	from debian:stable

	arg user=stephanie
	arg wd=/home/$arg/share/

	workdir $wd
	user root
	run apt update
	run apt dist-upgrade
	run apt install pymol

	run groupadd -r $user && useradd -r -m -s /bin/bash -g $user $user
	run chown -R $user:$user $wd

	user $user

# Build and create container
Move to the shared directory on the host and copy the `Dockerfile` there.

	docker build . --tag xapps
	docker run -it -e DISPLAY=$DISPLAY --net=host -v /host/path:/home/stephanie/share xapps

