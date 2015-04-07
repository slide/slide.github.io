---
title: 'IronPython and pyexpat - Part 1'
author: alex
layout: post
permalink: /2011/12/ironpython-and-pyexpat-part-1/
tweet_this_url:
  - http://is.gd/0e82A4
categories:
  - 'C#'
  - IronPython
---
A lot of the work needed to make IronPython into an even more useful piece of software (and citizen in the .NET world), there are still a number of C based Python modules that need to be ported to C#. If you look in the Modules directory under the Python 2.7.2 source code you'll see something like this:

<div id="attachment_72" class="wp-caption aligncenter" style="width: 907px">
  <a href="http://earl-of-code.com/wp-content/uploads/2011/12/cpython_modules_list.png"><img class="size-full wp-image-72" title="C Python Modules listing" src="http://earl-of-code.com/wp-content/uploads/2011/12/cpython_modules_list.png" alt="Listing of C Python .c modules" width="897" height="416" /></a>
  
  <p class="wp-caption-text">
    Listing of C Python 2.7.2 Source Modules directory
  </p>
</div>

Most of these files (with a few exceptions) are modules for Python that are implemented using the C extension feature of Python.

Here is the corresponding directory for IronPython with the C# implemented modules.

<div id="attachment_73" class="wp-caption aligncenter" style="width: 907px">
  <a href="http://earl-of-code.com/wp-content/uploads/2011/12/ironpython_modules_list.png"><img class="size-full wp-image-73" title="IronPython Modules List" src="http://earl-of-code.com/wp-content/uploads/2011/12/ironpython_modules_list.png" alt="Listing of IronPython C# modules" width="897" height="416" /></a>
  
  <p class="wp-caption-text">
    Listing of IronPython C# modules
  </p>
</div>

As you can see from these listings, there are several modules that still do not have an implementation for IronPython.

Porting modules can be tricky, there is not much in the way of documentation (ok, none at all) for implementing modules, but once you start into it, and get some info under your belt, I won't say anyone can do it, but its not as hard as people think.

One of the most requested modules was the zipimport module, which allows you to import Python modules from compressed archives. This includes modules compressed into egg files and just generic zip files. I recently submitted patches to IronPython to implement this functionality. The second most requested modules is the pyexpat module, which is used by the xml.parsers module. This blog series is going to go through the main steps that I take to port the pyexpat module from C Python to an IronPython module written in C#.

The next post in the series will be the initial setup of getting the source from github, adding the skeleton C# class for the pyexpat module and understanding how to run the unit tests that come with the IronPython source code.

If you are interested in more information about IronPython check out the CodePlex site at http://ironpython.codeplex.com.

