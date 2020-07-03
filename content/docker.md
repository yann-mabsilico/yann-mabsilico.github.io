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
in an ubuntesque way. The `NAMES` (plural, does it mean you can assign multiple names to a container?) column is obviously the last column of `docker ps -a`.

## Build an image and run it with tags

	docker build . --tag production
	docker run -it production

As mentioned above, `production` will appear in the `REPOSITORY` column of `docker images` while `latest` appears in the `TAG` column.
We can specify both `REPOSITORY` and `TAG` with `repository:tag`.

Note. Each call to `docker run -it production` will spawn a new container. So in a local setting (as opposed to say, azure)
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
- Case 1: your image has no tag (no `-t` option given at build).

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

Note that exiting the console in this case **will not** stop the container. We'll have to that manually:

	docker stop container_id

