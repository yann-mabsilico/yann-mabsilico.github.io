---
layout: default
title: azure
---

{: .title}
Azure

* TOC
{: toc}

# Storage mounting
[This page](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-how-to-mount-container-linux) explains how,
but the versions of `blobfuse` (`1.1.1` and `1.2.1` at the time of writing) contained in microsoft's repository **do not work**
(see [this issue](https://github.com/Azure/azure-storage-fuse/issues/428)).

Compiling the [github version](https://github.com/Azure/azure-storage-fuse/tree/v1.2.4) (`1.2.4` at the time of writing) worked.
The compilation required additional packages: `cmake, libcurl4-gnutls-dev, libgnutls28-dev, uuid-dev, libboost-filesystem-dev, libgcrypt20-dev, libfuse-dev`.

There is an extra step to make it work inside a `docker` container because we need access to the `fuse` device:

	docker run -it --device /dev/fuse --cap-add SYS_ADMIN --security-opt apparmor:unconfined (image_id|tag)

