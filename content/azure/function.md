---
layout: default
title: azure
---

{: .title}
Azure

* TOC
{: toc}

# Function apps
https://github.com/Azure/azure-functions-templates
Run-From-Zip is set to a remote URL using WEBSITE_RUN_FROM_PACKAGE or WEBSITE_USE_ZIP app setting. Deployment is not supported in this configuration.

## Creating the function
Azure -> Function App -> Add

When it's done, click on your newly created function and go to Settings -> Configuration, and delete `WEBSITE_RUN_FROM_PACKAGE`.

## Uploading files
### From a github action
See the `azure-poc-deploy` repository for an example of the organisation of files and for the github action that auto-deploys.
Note that the `requirements.txt` has to be at the root of the folder, and contains the package you want to install with `pip`.
An empty `requirements.txt` is fine, but even then the file has to be present.

For the use case of the repository, we chose an http trigger. The `hello` folder is actually a `python` module, and this is this module
that is called by the trigger, so it's important for the `__init__.py` and related files to be inside a directory.
Let's say we named it `directory_name`.

Once this is done, and if no `route` option is given in the `function.json` file, the function can be triggered by going to
`http://app_name.azurewebsites.net/api/directory_name`

See again the repository for an example of `route`.

### From the cli
It is very possible that the command

	az functionapp deployment source config-zip --subscription "Microsoft Azure Sponsorship" --resource-group batch-group --name mabstriggers --src app.zip

works **after** the `WEBSITE_RUN_FROM_PACKAGE` thingy is deleted. But I only tried it before! And it didn't work.

## Http triggers authentication
The authentication is set on each `function.json` with the `authLevel` field.

- `anonymous`: will work without authentication.
- `function`: requires a key set in the `code` field of the HTTP request, or provided as a HTTP `x-functions-keys` header.
	There are two sorts of keys. Some are `app` level, and some are `function` level.
	The `app` level keys are found on the corresponding `app` page, under the `App keys` of the `Functions` section.
	Both `_master` and `default` appear to work.
	To get to the `function` level keys, click on `Functions` in the `Functions` section, click on the function, and then go to `Function keys`.
- Other `authLevel`: I have not checked them yet.

