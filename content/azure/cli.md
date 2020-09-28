---
layout: default
title: azure
---

{: .title}
Azure

* TOC
{: toc}

# Je m'ai planté
Et j'ai créé des `user-level` credentials, qu'apparement on ne peut pas virer. Du coup j'ai généré un username et un mot de passe à coup de

	dd if=/dev/random bs=128 count=1 | md5sum

Faire

	az webapp development user show

montrera l'utilisateur, et on peut reset ça avec

	az webapp development user set --user-name xxx

# POC idea

- Create an input and an output storage.
- Add an azure function that triggers on input storage change. The trigger spawns a container (inside or outside a pool)
	and gives it the name of the file deposited in the storage. Then the container retrieves the file and runs something
	(`mabtope` at some point) on said file.
- Add some code to mabtope to send the result file to the output storage (same base name as the input file),
	and see how we do error handling (maybe a third storage?).
- Add an azure function that triggers on the output storage to give the results to WHOEVER.

# CLI
Before everything, log in to `Azure`. The operation is *environment-specific*, meaning terminal/user, and requires `root` (because of `docker`) :

		az login -u azure_login

And set the correct subscription:

		az account set --subscription "Subscription Name"

These commands can get long. Apparently, you can write a json file and give it to (all?) commands with the `--json-file` option.

[Random cheatsheet.](https://github.com/ferhaty/azure-cli-cheatsheet)

## Populate a registry
Source : [https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli)

- Log to the registry:

		az acr login --name registry_name

- Retag the image you want to upload:

		docker tag (image_id|tag) registry_name.azurecr.io/path/to/image:tag

- Push:

		docker push registry_name.azurecr.io/path/to/image

- Delete from the registry:

		az acr repository delete --name registry_name --image /path/to/image:tag

## Creating a container

- Manage resource groups:

		az group list
		az group create --name cli-containers --location francecentral
		az group delete --name cli-containers

- Manage credentials. You cannot create a container without credentials. If you are `Owner` of the azure account, you can get them with:

		az acr credential show --name registry_name

    If you are not `Owner`, ask an `Owner` to give you the credentials.

- Create, start, stop and delete a container:

		az container create --resource-group cli-containers --name cli-container --image registry_name.azurecr.io/samples/nginx --cpu 1 --memory 1 --ip-address Public
		az container start --resource-group cli-containers --name cli-container
		az container stop --resource-group cli-containers --name cli-container
		az container delete --resource-group cli-containers --name cli-container

When running container groups without long-running processes you may see repeated exits and restarts.
To resolve this problem, include a start command like the following with your container group deployment to keep the container running.
az container create --resource-group cli-containers --name cli-container --image registry_name.azurecr.io/samples/nginx --cpu 1 --memory 1 --ip-address Public --command-line 'tail -f /dev/null'

## Connecting to a running container

	az container exec --resource-group cli-containers --name cli-container --exec-command '/bin/bash'

Got a certificate error with this ^

## Batch
https://docs.microsoft.com/en-us/azure/batch/batch-cli-get-started
https://docs.microsoft.com/en-us/azure/batch/scripts/batch-cli-sample-manage-linux-pool
https://docs.microsoft.com/en-us/azure/batch/batch-docker-container-workloads

You create a batch account, inside which there is a (batch) pool of VMs (not containers). These VMs should (for our purposes) be
able to run containers, which essentially reduces the choice of the VM image (assuming we go with azure predefined ones) to

- centos-container
- ubuntu-server-container
- centos-container-rdma
- ubuntu-server-container-rdma

`RDMA` means *Remote direct memory access* and is apparently used to have a memory that is common to multiple nodes. Could be useful.
After that, running a job means deploying a container on the VMs of the pool.

- New resource group:

		az group list
		az group create --name batch-group --location francecentral

- New batch account (no dashes allowed in the name):

		az batch account create --subscription "Sub Name" --resource-group batch-group --name batchaccount --location francecentral

	There is also an option for auto-storage. I'm not sure what it does.

- Login to the new batch account

		az batch account login --subscription "Sub Name" --resource-group batch-test --name batchaccount

- New pool:

		az batch pool create --account-name batchaccount --id batch-pool --image "microsoft-azure-batch:centos-container:7-7:latest"\
			--vm-size "Standard_A1" --target-dedicated-nodes 2 --node-agent-sku-id "batch.node.centos 7"

	I don't know how to list the images that offer container deployment (the ones I listed above are the Microsoft prepared ones).
	Once you have your image name (`centos-container`), getting a compatible `--image` field is done with:

		az vm image list --all --offer centos-container

	And OBVIOUSLY, you take the `urn` value. No idea how to get the proper `--node-agent-sku-id`. Got this one from
	https://docs.microsoft.com/en-us/azure/batch/batch-linux-nodes

**--- BOOM ---**

Found a [post from 2018](https://stackoverflow.com/questions/53677297/container-task-settings-for-azure-batch-task-using-cli)
saying it is impossible to create a container-task from the CLI, but we could use some `json`. While not saying this explicitly, the
[documentation](https://docs.microsoft.com/en-us/azure/batch/batch-docker-container-workloads)
doesn't give any example of how to do this, instead resorting to `python` or `C#` scripts. So we move on to...

# Python
Apparently, you need to register an azure application in order to connect. So I did that.
See [here for explanations](https://www.inkoop.io/blog/how-to-get-azure-api-credentials/).

Also, you need to give your application the proper permissions.

Jobs and tasks IDs are unique over all pools.

Instruction to pull the container image from your azure registry: [here](https://www.muspells.net/blog/2018/11/azure-batch-task-in-containers/)

