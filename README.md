# Frequently Asked Questions

As stewards of the official images and maintainers of many images ourselves, we often see a lot of questions that surface repeatedly. This repository is an attempt to gather some of those and provide some answers!

## Table of Contents

<!-- AUTOGENERATED TOC -->

<!-- AUTOGENERATED TOC -->

## General Questions

### What do you mean by "Official"?

This question is so common it's answered in our primary repository!

See [the answer in the readme of the `github.com/docker-library/official-images` repository](https://github.com/docker-library/official-images#what-do-you-mean-by-official).

### How are images built? (especially multiarch)

Images are built via a [semi-complex Jenkins infrastructure](https://doi-janky.infosiftr.net/), and the sources for much of that can be found in [the `github.com/docker-library/oi-janky-groovy` repository](https://github.com/docker-library/oi-janky-groovy).

The infrastructure is a combination of machines provided by Docker, Inc. and several generous donors (for non-`amd64` architectures):

-	`arm32vN`: [WorksOnArm](https://github.com/WorksOnArm/cluster/issues/7)
-	`arm32v6` (selected images): QEMU on Tianon's personal machine; see:
	-	https://github.com/docker-library/busybox/pull/41
	-	https://github.com/docker-library/golang/issues/196
	-	https://github.com/docker-library/memcached/issues/25
	-	https://github.com/docker-library/postgres/issues/420#issuecomment-370033903
	-	https://github.com/docker-library/redis/issues/137
-	`arm64v8`: [Linaro](https://www.linaro.org/)
-	`ppc64le`, `s390x`: [IBM](https://www.ibm.com/)

At a high-level, the image publishing process looks something like this:

1.	images are built on a machine relevant to their architecture
2.	architecture-specific images get pushed to the respective architecture-specific Docker Hub namespace (`amd64/xxx`, `arm64v8/xxx`, `s390x/xxx`, etc)
3.	a manifest list is created for `library/xxx` from the list of architecture-specific artifacts ([caveat docker-library/official-images#3835](https://github.com/docker-library/official-images/issues/3835))

### What is `bashbrew`? Where can I download it?

The `bashbrew` tool is one built by the official images team for the purposes of building and pushing the images. At a very high level, it's a wrapper around `git` and `docker build` in order to help us manage the various `library/xxx` files in the main official images repository in a simple and repeatable way (especially focused around using explicit Git commits in order to achieve maximum repeatability and `Dockerfile` source change reviewability).

The source code is currently found in [the `bashbrew/` subdirectory](https://github.com/docker-library/official-images/tree/master/bashbrew) of [the `github.com/docker-library/official-images` repository](https://github.com/docker-library/official-images). Precompiled artifacts (which are used on the official build servers) can be downloaded from [the relevant Jenkins job](https://doi-janky.infosiftr.net/job/bashbrew/lastSuccessfulBuild/artifact/bin/).

## Image Building

### Why do so many official images build from source?

The tendancy for many official images to build from source is a direct result of trying to closely follow each upstream's official recommendations for how to deploy and consume their product/project.

For example, the PostgreSQL project publishes (and recommends the use of) their own official `.deb` packages, so [the `postgres` image](https://hub.docker.com/_/postgres/) builds directly from those (from http://apt.postgresql.org/).

On the flip side, the PHP project will only officially support users who are using the latest release (https://bugs.php.net/, "Make sure you are using the latest stable version or a build from Git"), which the distributions do not provide. Additionally, their "installation" documentation describes building from source as the officially supported method of consuming PHP.

One common result of this is that Alpine-based images are almost always required to build from source because it is somewhat rare for an upstream to provide "official" binaries, but when they do they're almost always in the form of something linked against glibc and as such it is very rare for Alpine-compatible binaries to be published (hence most Alpine images building from source).

So to summarize, there isn't an "official images" policy one way or the other regarding building from source; we leave it up to each image maintainer to make the appropriate judgement on what's going to be the best representation / most supported solution for the upstream project they're representing.

### `HEALTHCHECK`

Explicit health checks are not added to official images for a number of reasons, some of which include:

-	many users will have their own idea of what "healthy" means and credentials change over time making generic health checks hard to define
-	after upgrading their images, current users will have extra unexpected load on their systems for healthchecks they don't necessarily need/want and may be unaware of
-	Kubernetes does not use Docker's heath checks (opting instead for separate [`liveness` and `readiness` probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/))
-	sometimes things like databases will take too long to initialize, and a defined health check will often cause the orchestration system to prematurely kill the container ([docker-library/mysql#439 for instance](https://github.com/docker-library/mysql/issues/439))

The [docker-library/healthcheck repository](https://github.com/docker-library/healthcheck) is to serve as an example for creating your own image derived from the prototypes present. They serve to showcase the best practices in creating your own healthcheck for your specific task and needs.
