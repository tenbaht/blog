+++
title = "Compiling GrblControl in a Container"
date = 2020-10-07T10:47:40+02:00
draft = false
tags = ["grblControl","gcode","container"]
categories = ["software", "cnc"]
+++

![Compiling GrblControl in a Container](/images/compiling-candle.png)

Using a container for a build environment can help to avoid cluttering up
the filesystem of the local machine. This approach can be useful for
one-time builds with lots of build dependencies which are not used for any
other work.

Since there is no precompiled 64 bit binary for
[Candle](https://github.com/Denvi/Candle) (formerly known as grblControl) I
had to compile it myself. But since I never use the QT libraries otherwise
it would pull in more than 600MB of libraries and header files which I would
have to clean up afterwards in order to keep my system as lean as possible.

A perfect opportunity to find out why containers might be useful!
I am using podman-rootless, but plain old docker would work exactly the same
way. Just replace `podman` with `docker` in the following commands.

<!--more-->


## Setting up the build environment

Download and unpack the Candle source code somewhere.

	git clone https://github.com/Denvi/Candle.git

Start an interactive bash session on an Ubuntu in a container and give it
access to the directory with the Candle sources:

	podman run -v /path/to/candle/dir:/build -it ubuntu bash

The magic unfolds: A quick download (72MB), some unpacking et voilà: A bash
root prompt, running on a minimal ubuntu 20.04 - ready for action! `ls
/build` should show the Candle source directory.


## Build process inside the container

Now I am basically following [Naisema's
guide](https://naisema.blogspot.com/2017/11/install-candle-on-ubuntu-64bit-machines.html)
inside this container environment. These commands install all the required
dependencies. More than 600MB in my case. `gksu` seem to be merged into one
of the other QT files and is no longer required or even available:

	apt update
	apt install make gcc g++
	apt install qt5-default qt5-qmake libqt5serialport5-dev

It asks about the timezone. It doesn't matter, any answer will do.

	cd /build
	qmake
	make

Done! The binary is generated in `src` directory. We can leave the
containered bash:

	exit



## Install the binaries

Strangely, there is no `make install` target and we have to do it ourself:

	install -Dt src/Candle /usr/local/bin
	install -m644 -D Candle/src/images/candle_256.png /usr/local/share/app-install/icons/candle.png

I am not sure about the content of `src/translations`. They might be useful,
but it works without them.

Finally, save this as `/usr/local/share/applications/candle.desktop`:

	[Desktop Entry]
	Name=Candle
	GenericName=Candle
	Comment=CNC controller software
	Exec=/usr/local/bin/Candle
	Icon=/usr/local/share/app-install/icons/candle.png
	Terminal=false
	Type=Application
	Categories=Development;Electronics;




## Deleting the container files

Find out the name of the containered system (last column):

	podman ps -a

And delete all the files:

	podman rm [container-name]

Done! A clutter free system again!


## File attachments

[pre-compiled 64 bit binary](/uploads/candle.tgz) for Ubuntu 20.04/Mint 19

