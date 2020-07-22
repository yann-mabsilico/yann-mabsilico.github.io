# Azure CLI
# Peupler un registry (depuis sa machine)
Source : [https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli)

- Log in to azure. The operation is *environment-specific*, meaning terminal/user, and requires `root` (because of `docker`) :

		az login -u azure_login

- Log to the registry:

		az acr login --name registry_name

- Retag the image you want to upload:

		docker tag (image_id|tag) registry_name.azurecr.io/path/to/image:tag

- Push:

		docker push registry_name.azurecr.io/path/to/image

- Delete from the registry:

		az acr repository delete --name registry_name --image /path/to/image:tag

# Creating a container

- Set the correct subscription:

		az account set --subscription "Subscription Name"

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

