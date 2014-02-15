---
layout: post
title: "Downloading API Documentation with WGET"
date: 2013-05-22 22:18
comments: true
categories: 
- WGET
- Programming
- API
- Documentation
---
When I was trying to learn Google Web Toolkit (GWT), I wanted to download the entire API documentation for my reference and offline viewing. One problem that I encountered is that unlike JDK, GWT does not provide downloads for their API documentation. So an idea come up to me that maybe I should use something like CURL or WGET to get their documentation. So, I googled if that was possible, and found [this amazing Linux Journal Tip](http://www.linuxjournal.com/content/downloading-entire-web-site-wget).
<!-- more -->

When I was trying out the command directly, WGET gives me an error. So I checked every options from Linux Journal against WGET's man page and found that `--html-extension` has been changed to `--adjust-extension` since version 1.12. So, here's the complete command:

	$ wget \
		--recursive \
		--no-clobber \
		--page-requisites \
		--adjust-extension \
		--convert-links \
		--restrict-file-names=windows \
		--domains google-web-toolkit.googlecode.com/ \
		--no-parent \
			google-web-toolkit.googlecode.com/svn/javadoc/latest

And here's the options description from Linux Journal:

- `--recursive`: download the entire Web site.  
- `--domains google-web-toolkit.googlecode.com`: don't follow links outside google-web-toolkit.googlecode.com.  
- `--no-parent`: don't follow links outside the directory tutorials/html/.  
- `--page-requisites`: get all the elements that compose the page (images, CSS and so on).  
- `--html-extension`: save files with the .html extension.  
- `--convert-links`: convert links so that they work locally, off-line.  
- `--restrict-file-names=windows`: modify filenames so that they will work in Windows as well.  
- `--no-clobber`: don't overwrite any existing files (used in case the download is interrupted and resumed).  

After that, it will take some time for it to complete. Go get yourself a cup of tea or do something else, it's probably longer than downloading a single tarball documentation.

##Make a Shellscript##
People like me couldn't remember such a long options, so, I created a simple shell script to assist me in updating the documentation, fetching other documentation, and downloading a website:

	#!/bin/sh

	wget \
		--recursive \
		--no-clobber \
		--page-requisites \
		--adjust-extension \
		--convert-links \
		--restrict-file-names=windows \
		--domains $(echo $1 | sed 's!http://!!' | cut -d/ -f1) \
		--no-parent \
			$1

Of course you can put it in a shell function as well if you like, but I prefer shell scripts because it's easier to manage.

##REFERENCES##
1\. [Linux Journal Article on WGET](http://www.linuxjournal.com/content/downloading-entire-web-site-wget)
