---
layout: default
title: ssh github 2fa
---

{: .title}
Setting up ssh access to github

* TOC
{: toc}

# Generate a ssh key pair

	ssh-keygen -t rsa -f key_name

creates 2 files: `key_name` (private key) and `key_name.pub` (public key).
Put the private key in `~/.ssh/`.

# Give your public key to github
Upper right corner, click the profile photo, then `Settings`.
In the left sidebar, click `SSH and GPG keys`, then click `New SSH key or Add SSH key`.
Copy / paste the public key in the `Key` field, click `Add SSH key`.

# Connecting to github with the ssh key
Copy these lines in the `~/.ssh/config` file:

	Host github
		Hostname github.com
		User git
		IdentityFile ~/.ssh/key_name

then

	git clone github:mabsilico/sequence_database_scripts

