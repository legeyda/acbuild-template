


acbuild-template
================

Write declarative acbuild-scripts
and build/install/run images from them 
without need of any auxiliary boilerplate code.



Motivation
----------

I like that acbuild follows the unix-ideology to be flexible and composable tool.
Unlike Dockerfile, acbuild can be built into scripts or makefiles 
to implement sophisticated image-bilding behaviour, 
such that would be impossible to make with Docker.

However, there are majority of cases,
where just very simple behaviour is needed, e.g.: 
(1) get base image, (2) make some simple modification to it
and (3) get the result.

In those cases the declarative nature of Dockerfiles
seems to be superior to acbuild imperative approach.
Dockerfile is all-in-one entity with simple syntax,
which is self-explaining for reader.
It is enough to look at Dockerfile to have a 
grasp of what is going on when image is built.

On the other hand, acbuild needs lots of 
auxilliary imperative scripting to execute.
As a writer, I am to write some build scripts, 
or a makefile in which i should implement lots
of things like build artifact management, version propagation etc.
As a reader, I find it hard to read such implementations, 
since the code is split among multiple files.

I like the support for scripts added in the recent versions of acbuild,
which allows to make more elegant build systems.
But there are problems with scripts.
For example, suppose i want to store version number only in one place in the project,
to avoid mistakes when building releases. 
I store version number in my makefile, 
so i need some kind of templating to pass version number to script.
Again i need boilerplate code.

acbuild-template implements simple idea of using 
variable placeholders instead of hard-coded values in acbuild scripts
to make executing them more flexible.



Introduction
------------

1.	Write acbuild script named `acbuild-script`, using variable placeholders to identify values which can vary:

		begin --build-mode={{mode}}
		set-name example.com/nginx
		label add version 1.2.3
		run --insecure={{insecure}} --engine={{engine}} -- bash -c 'touch /var/lib/rpm/* && yum install nginx && yum clean all'
		set-exec -- nginx -g 'daemon off;'
		write --overwrite {{output}}

2.	With a single command build/install image or run container from the script using default values, which acbuild-template substitutes for you:
	
		$ cd path/to/script
		$ acbuild-template build --name-only
		/tmp/tmp.duwasSaaLk/example_com_nginx-1.2.3-linux-amd64.aci
		$ acbuild-template install --name-only
		sha512-0a1cb92dc276b0f9bedf87981e61ecde
		$ acbuild-template run
		...

Of course, values can be customized, eg: 

	ACBUILD_RUN_INSECURE=true ACBUILD_RUN_ENGINE=chroot ACBUILD_IMAGE_BASE_DIR=/tmp/acbulid-cache acbuild-template run acbuild-script.template



Usage
-----

	  acbuild-template help
	  acbuild-template apply     [<template>[target script]]
	  acbuild-template build     [-n|--name-only] [<template>[<target image>]]
	  acbuild-template install   [-n|--name-only] [<template>]
	  acbuild-template run       [<template>] [<rtk run args> ... ]

	Options:
	  -h --help                print this message
	  -n --name-only           build:   print nothing except resulting image full name
	                           install: print nothing except installed image hash

	Environment:

	  ACBUILD_SCRIPT           script to process (./acbuild-script by default)
	  ACBUILD_IMAGE_DIR_NAME   directory to save resulting image
	  ACBUILD_IMAGE_MODE       build mode, appc or oci
	  ACBUILD_IMAGE_VERSION    label add os version (snapshot by default)
	  ACBUILD_IMAGE_OS         label add os value (current by default)
	  ACBUILD_IMAGE_ARCH       label add os value (current by arch)
	  ACBUILD_IMAGE_EXT        image file extension, default aci
	  ACBUILD_RUN_INSECURE     run --insecure=value, defaul false
	  ACBUILD_RUN_ENGINE       run --engine=value
	  ACBUILD_IMAGE_BASE_NAME  image base file name, default {{name}}-{{version}}-{{os}}-{{arch}}.{{ext}}
	  ACBUILD_IMAGE            image file full name
	  ACBUILD_CACHE_DIR        directory to store temporary files

	  ACBUILD                  acbuild command, default acbuild itself
	  ACBUILD_OPTS

	  RKT                      rkt command, default rkt itself
	  RKT_OPTS
	  RKT_FETCH_OPTS
	  RKT_RUN_OPTS
