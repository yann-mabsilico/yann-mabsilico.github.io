---
layout: default
title: docker
---

{: .title}
Docker - Cheat sheet

# Commands
## Listing
- List images:

		docker images

- List containers:

		docker ps -a

	(Did you guess `docker containers`?)

## Interlude
A quick section to highlight the consistency of docker's command line interface.

### Image tags
The first two columns of `docker images` are `REPOSITORY` and `TAG`. In the docker documentation and in this document, the `tag` of an image
refers to the pair `REPOSITORY:TAG`. We can often omit the `:TAG` part, in which case the tag we give
(for example when retaging, see [Retag an image](#retag-an-image)) will go into the `REPOSITORY` column and the `TAG` column will default to `latest`.

When building an image, if we don't specify a tag (again, see [Build an image and run it with tags](#build-an-image-and-run-it-with-tags)),
both `REPOSITORY` and `TAG` will default to `<none>`.

### Container names
Containers don't have tags, they have names. Upon being run for the first time, a name for the container is generated automatically
in an ubuntuesque way. The `NAMES` (plural, does it mean you can assign multiple names to a container?) column is obviously the last column of `docker ps -a`.

## ID hashes
The result of listing commands contain ids for both images and containers. These ids are twelve character long hexadecimals.
When using these ids (in the commands below), you (unless extreme circumstances) don't need to type all twelve characters.
All you need to type is enough characters (starting from the left) to make it uniquely identifying.
As an example, if only one container id starts with *2*, then we can simply use *2* instead of the full id to identify this container.

## Build an image and run it with tags
A build requires a `Dockerfile`. Refer to the [Dockerfile](#dockerfile) section. If no `Dockerfile` is specified,
the `build` command will look in the current working directory. One may specify a different location for the `Dockerfile` with the `-f` option.

	docker build . --tag production
	docker run -it production

The dot is the context directory (and could obviously be any directory). See section [Understanding the context](#understanding-the-context) for an explanation.

As mentioned above, `production` will appear in the `REPOSITORY` column of `docker images` while `latest` appears in the `TAG` column.
We can specify both `REPOSITORY` and `TAG` with `repository:tag`.

Notes.

- If an image `X` with the `production` tag existed before the `build` command, the tag of image `X` will be removed and default back to `<none>:<none>`.
- Each call to `docker run -it production` will spawn a new container. So in a local setting (as opposed to say, azure)
	this should only be done once, after which we can simply reuse the container (see [Restart a container](#restart-a-container)).

## Remove image
Exited containers still use the image because reason. If you try to remove an image that's still in use with:

	docker rmi image_id

it'll tell you the image can't be deleted because it's in use, do either

	docker rm container_id
	docker rmi image_id

or

	docker rmi -f image_id

If your image only has one tag, you can remove the image by calling the tag:

	docker rmi -f repository:tag

or simply

	docker rmi -f repository

if `tag` is `latest`.

If your image has several tags, removing the image will remove all of its tagged versions.

## Remove all images and containers

	docker rmi `docker images -q`
	docker rm `docker ps -a -q`

## Retag an image
- Case 1: your image has no tag (no `--tag` option given at build).

		docker tag image_id tag

	The image with no tag does not exist anymore.

- Case 2: you have an image with a tag.

		docker tag (old_tag|image_id) new_tag

	This will create a new image with `new_tag`. Both `old_tag` and `new_tag` will point to the same image.
	If we want only the new tag to remain, we have to remove the old one.

		docker rmi -f old_tag

## Commit changes
Changes made inside a container **do not change the image**. Unless you tell it to manually.

- Close the container and get its id with

		docker ps -a

- Then commit your changes with

		docker commit container_id tag

If the tag exists, only this tag will be affected, and you'll see a change in the image id for this tag:
if you had two tags pointing to the same image id, they will now point to different ids.
If the tag does not exist, a new image will be created with this tag.

## Restart a container
After exiting a container started with the image (like above) the container will be in the `Exited` status (see `docker ps -a`).
It can be started again with

	docker start container_id

Nothing seems to happen, but a `docker ps -a` will show that the container is now running. We can reenter it with

	docker exec -ti container_id /bin/bash

Note that exiting the console in this case **will not stop** the container. We'll have to do that manually:

	docker stop container_id

## Copy files from the host to the container

	docker cp /local/path container_id:/container/path

# Dockerfile

Let's use this example script to explain the format and keywords.

	from debian:stable

	workdir /root/
	copy .vimrc /root/
	add mabtope.tar .

	run apt-get update
	run apt-get install --yes python3
	run python3 mabtope/_docker_/install.py

## `from`
The `from` keyword takes a tag as parameter. It is meant to say: take the `from` image as the base of the image we want to build.
The tag is first searched locally, that is, you can build an image starting from an image you already created.
If the local search fails, images are pulled from the web (somewhere). In this case, `debian:stable` exists on the web and
is the image of a (very) minimal debian stable image.

## `workdir`
The `workdir` keyword has two functions:

- subsequent commands of the `Dockerfile` will be executed from this directory,
- any terminal opened on the containers based on this image will start in this directory.

## Understanding the context
The context is the directory given as a parameter of the `docker build` command. When `build` is called, everything within the context,
that is, the full directory, is copied into a temporary folder that will be used by `docker` to copy files over to the image.
That copy is super important, because it means that `docker` **only knows what is inside the context**. Most notably, if `./`
is given as context, then docker doesn't know what `../` is.

In case anyone is wondering, the path to the `Dockerfile` has absolutely no influence on the context.

## `copy` and `add`
Both keywords take two parameters:

- a *relative* path, with the context as the base, that is within the `context` (again, no path outside of the context is allowed),
- a path in the image.

The difference between the two keywords is that the first parameter of `add` is an archive (I know `tar` and `tar.gz` work, the others probably also do)
that will automatically be extracted inside the image.

## `run`
Runs any command within the image.

## Note
The above `Dockerfile` is used for `mabtope`. It is essentially the minimal file that allows calling a proper installation script (`install.py`).
I'd say it's a good strategy.

