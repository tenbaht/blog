+++
title = "Installing Onetab in Palemoon"
date = 2020-07-24T14:18:20+02:00
tags = ["www"]
categories = ["network"]
+++

![Installing Onetab in Palemoon](/images/titelbild.jpg)
{{< figure src="/images/installing-onetab-in-palemoon.png" caption="Onetab 1.9 running with palemoon 28.9.0.2" >}}

The amazing and absolutely essential onetab extension is only available for
firefox and chrome, not for the stripped-down firefox version palemoon. But
with the help of archive.org and a text editor it is possible to install an
older (but fully functional) version. A pre-patched add-on file is included
in this posting.
<!--more-->

Palemoon still uses the old xpi file format for add-ons. It never did the
switch to the crx format that chrome and more versions of firefox use. But
it is very easy to patch an old xpi file for use with palemoon.


## Pre-patched files

If you trust me enough just click here to install this pre-patched version
of the 1.9 files:
[Patched version 1.9 for palemoon](static/uploads/palemoon-onetab-1.9.xpi)

And for reference the original xpi file as from archive.org/mozilla.org:
[Original version 1.9 for firefox](static/uploads/onetab-1.9-fx.xpi)



## Patching the add-on archive

If you are afraid of unauthorized modifications in my version here is how to
patch it yourself.

The older versions of onetab in the xpi format are still available at
[archive.org](https://web.archive.org/web/20150922055958/https://addons.mozilla.org/en-US/firefox/addon/onetab/versions/).

Right click on the [add to firefox](https://web.archive.org/web/20150922055958/https://addons.mozilla.org/firefox/downloads/file/267884/onetab-1.9-fx.xpi?src=version-history)
button to save the xpi file.

unzip the downloaded file in an empty directory:

	mkdir onetab
	cd onetab
	unzip ~/Downloads/onetab-1.9-fx.xpi

Edit install.rdf by replacing the firefox id ec8030f7-c20a-464f-9b0e-13a3a9e97384
with the palemoon id 8de7fcbb-c55c-4fbe-bfc5-fc555c87dbc4:

	sed -i s/ec8030f7-c20a-464f-9b0e-13a3a9e97384/8de7fcbb-c55c-4fbe-bfc5-fc555c87dbc4/ install.rdf

The result should read like this:

    <em:targetApplication>
      <Description>
        <em:id>{8de7fcbb-c55c-4fbe-bfc5-fc555c87dbc4}</em:id>
        <em:minVersion>21.0</em:minVersion>
        <em:maxVersion>29.0a1</em:maxVersion>
      </Description>
    </em:targetApplication>

repack the xpi file:

	zip -r ../palemoon-onetab-1.9.xpi .

Now open this file in palemoon to install the addon:

	palemoon ../palemoon-onetab-1.9.xpi
