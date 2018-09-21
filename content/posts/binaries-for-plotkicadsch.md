+++
title = "Binaries for plotkicadsch"
date = 2018-09-21T22:29:17+02:00
draft = false
tags = ["kicad"]
categories = ["electronics"]
+++

## Installing the precompiled binaries

Download my binaries for [plotkicadsch](https://github.com/jnavila/plotkicadsch)

* [for Mint 19/Ubuntu 18.04, 64 bit](/uploads/plotkicadsch_v0.3.0-21-ga5b939b.tgz)


Unpack the file to a convinient location like `/usr/local/bin`:

	sudo tar xvzf plotkicadsch_v0.3.0-21-ga5b939b.tgz -C /usr/local/bin

Now it can be used to compare the current state to the last git version and
display the result e.g. with firefox (see project website for more usage
information):

	plotgitsch -ifirefox


## Compiling from source

`plotkicadsch` by [Jean-Noël Avila](https://github.com/jnavila) is an
amazing tool to visualize differences on schematics between different
versions of a project.

The only problem is that it is written in the pretty exotic language ocaml
and that there are no precompiled binaries available. Compiling it from
source is well documented on the github page, but it requires a *HUGE*
amount of harddisk space to compile: 1.6 GByte in the home directory plus the
ocaml installation.

For compilation you need ocaml, opam, libgmp and rsvg-convert:

	sudo apt install opam ocaml libgmp-dev librsvg2-bin

Download and compile as mentioned on the project page. This results in 1.6
GByte (!!!) of data in your home directory in `~/.opam`. Luckily, only three
files need to be installed permanently:

	sudo cp -av ~/.opam/4.06.0/bin/plot*sch git-imgdiff /usr/local/bin/


## Cleaning up

As long as you don't plan on compiling every new version of plotkicadsch as
it is published, the compiler installation and all the files in `~/.opam` are
not needed anymore anymore and can be wiped off the disk:

	sudo apt purge opam ocaml libgmp-dev librsvg2-bin
	sudo apt autoremove
	rm -r ~/.opam

Now the system should be in the same state as before. Docker would the
perfect tool to automate this build process, but I didn't spend
enough effort to make that work. Anyone?
