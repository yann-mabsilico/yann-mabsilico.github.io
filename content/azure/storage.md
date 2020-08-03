---
layout: default
title: azure
---

{: .title}
Azure

* TOC
{: toc}

# Storage mounting
[This page](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-how-to-mount-container-linux) explains how, but:

- the versions of `blobfuse` (`1.1.1` and `1.2.1` at the time of writing) contained in microsoft's repository **do not work**
	(see [this issue](https://github.com/Azure/azure-storage-fuse/issues/428)).
- the path of the `--tmp-path` option **must be absolute**, otherwise a surprise `<no name>` directory will be created when we write files
	(apparently no need for the other paths to be absolute).

Compiling the [github version](https://github.com/Azure/azure-storage-fuse/tree/v1.2.4) (`1.2.4` at the time of writing) worked.
The compilation required additional packages:

	cmake g++ libboost-filesystem-dev libboost-thread-dev libcurl4-gnutls-dev libfuse-dev libgcrypt20-dev libgnutls28-dev pkg-config uuid-dev 

There is an extra step to make it work inside a `docker` container because we need access to the `fuse` device:

	docker run -it --device /dev/fuse --cap-add SYS_ADMIN --security-opt apparmor:unconfined (image_id|tag)

To unmount, no need for `fuse`!

	umount /path/to/folder/

