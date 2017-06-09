
acbuild-template
================

Write declarative acbuild-scripts
and build/install/run images from them 
without need of boilerplate code.


Motivation
----------

I like that acbuild follows the unix-ideology to be flexible and composable tool.
Compared to Dockerfile, acbuild can be used in build scripts or makefiles to
implement sophisticated image-bilding behaviour, 
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
grasp of what is going on with that image building process.

On the other hand, acbuild needs lots of 
auxilliary imperative scripting to build image.
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


Usage
-----

1.	Write acbuild script template, using variable placeholders:

		begin --build-mode={{mode}}
		set-name example.com/nginx
		label add version 1.2.3
		run --insecure={{insecure}} --engine={{engine}} bash -c 'touch /var/lib/rpm/* && yum install nginx && yum clean all'
		write --overwrite {{output}}

2.	Run container from script, accepting default values for placeholders:
	
		acbuild-template run acbuild-script.template --volume

Or run container with customized values: 

		ACBUILD_RUN_INSECURE=true ACBUILD_RUN_ENGINE=chroot ACBUILD_IMAGE_BASE_DIR=/tmp/acbulid-cache acbuild-template run acbuild-script.template --volume