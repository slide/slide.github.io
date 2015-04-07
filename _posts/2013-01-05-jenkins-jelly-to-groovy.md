---
title: 'Jenkins - Jelly to Groovy'
author: alex
layout: post
permalink: /2013/01/jenkins-jelly-to-groovy/
categories:
  - Jenkins
  - Python
---
At work we use [Jenkins][1] for our continuous integration setup. As I have mentioned [previously ][2]I really, really like Jenkins (I blogged about Hudson previously, but we moved with the fork to Jenkins since it is more community driven).

I took over as the maintainer for the [email-ext][3] plugin, which allows you to configure the emails sent for failures and other build results to a much higher level than the default Mailer. You can have different triggers for different statuses, you can include various pieces of information in your email templates. You can even use scripts to generate the emails. See the wiki page above if you are interested in more information.

Jenkins uses MVC for displaying web pages and interacting with the system. You create a view template in either [Jelly][4], which is an "executable" XML format, or [Groovy][5] which is a scripting language for the Java Virtual Machine (JVM). The Jelly format is VERY painful to try and debug what is going wrong if you have something going wrong. Errors are not easy to track down and it is VERY painful to do some things (like call methods on objects and define variables and conditionals and...well pretty much everything). Groovy, on the other hand, it very nice to work with. You have basically a full scripting language to use to your advantage. Conditionals, object creation, variables are all just as easy as if you were writing a simple script (which you are!).

I wanted to start migrating the email-ext plugin to use Groovy views, because I think it gives a lot of power when trying to do things with the Jenkins API. I hand ported one view and it didn't really take that long, but as most software people realize at some point when dealing with XML, the computer could be doing this for me! I spent about 20 minutes or so writing this initial version of a Jelly to Groovy converter. For very simple views, it works great. Feel free to fork it on GitHub and send me a pull request with updates. Hopefully it will be useful to someone else.

<https://github.com/slide/jelly2groovy>



 [1]: http://jenkins-ci.org "Jenkins Continuous Integration "
 [2]: http://earl-of-code.com/2010/07/hudson/
 [3]: https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin
 [4]: http://commons.apache.org/jelly/
 [5]: http://groovy.codehaus.org/