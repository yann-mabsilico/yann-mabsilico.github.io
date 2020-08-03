---
layout: default
title: azure
---

{: .title}
Azure

* TOC
{: toc}

# Storage containers
When an app is created, you're asked to create a storage account for this app. Inside this storage account are what Azure **very** stupidly calls *containers*.
I will refer to them as *storage container*, and keep the lone *container* for docker.
These storage containers act like folders, and can be created by pressing the `+Container` button of the `Blob service/Containers` menu of the storage account page.

# Setting the blob trigger
[Documentation Microsoft](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob-trigger?tabs=python).

An example.

	{                                                               
		"scriptFile": "__init__.py",            
		"disabled": false,        
		"bindings": [
			{   
				"name": "blob",
				"type": "blobTrigger",
				"direction": "in",
				"path": "input/{brick}.{name}",
				"connection":""                                    
			}       
		]
	}

- `name`: the name of the variable given to the script. The name given here **must** match with the name of the variable given to the `main` function.
- `connection`: this is where you put the credentials to access the storage. If left blank, the `AzureWebJobsStorage` variable will be used.
	The doc says you may simply put the remainder of the name and it will automatically be prefixed by `AzureWebJobs`. Does this mean you can't
	choose an environment variable whose name doesn't start with `AzureWebJobs`?
- `path`: here, `input` is the storage container and what is after the slash is the name of the file. So you can actually condition
	the trigger on the name: in my example, the name needs to contain one dot. What's inside the braces are members of the variable
	passed to the script, so `blob.brick` and `blob.name` will be filled.

# Accessing the folders
When mounting a storage account, you must provide three parameters:
- the storage account name,
- a storage account key,
- a storage container name.

When you are using the storage from an app, the storage account name and key are already contained in the `AzureWebJobsStorage` environment variable
(found in the `Settings/Configuration` menu of your app page).

# Accessing the batch account
Since we want to launch jobs and tasks on a pool of a batch account, we need the batch account credentials, which is just a name and a key.
I just set the key in the `_AzureBatchAccountKey` environment variable of the app. It is retrieved from the python script with

	os.environ['_AzureBatchAccountKey']

# Passing environment variables to docker
The docker container needs to be able to mount storage folders, so it needs the `AzureWebJobsStorage` environment variable. Just give this
to the container run options:

	-e AzureWebJobsStorage

The docker container will also need to know the name of the file that was created in the storage:

	-e _AzureInputFileName=fileName

where the file name is retrieved in the python script from the `blob` variable.

# Mounting the storage containers inside docker
The `AzureWebJobsStorage` variable is a http request string.

# ASONHTUSARCEPGSA.N,RGP SNTEO.H
Les storage v1 ne sont pas reconnus comme 'resource' du resource-group lors de la création d'un event grid.
Ptet que c'est pour ça que le trigger ne marche pas (apparement non, ça ne marche toujours pas).
ET BIEN SUR LES STORAGE PAR DEFAULT QUAND TU CREE UNE APP SONT EN V1. @#!§

