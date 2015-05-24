---
title: 'Groovy 'def' Jam'
author: alex
layout: post
permalink: /2013/02/groovy-def-jam/
categories:
  - groovy
  - Jenkins
  - open source
---
As a way to blow off some steam, I like to contribute to [open source software][1]. You might ask why, as a software developer who spends his entire day writing code would I want to spend my free time writing more software. I honestly don't know the answer to that question. I find something enjoyable in giving something back, or something along those lines.

Anywho, one of the projects that I contribute to and have talked about on here before is the [Jenkins][2] continuous integration server. Jenkins uses an MVC model for displaying webpages and interacting with the API of the application, the views are created by using [Jelly][3]. I will be completely honest, I hate creating and using Jelly views. Whoever thought up "executable XML"...

I found out that you could also do views using [Groovy][4], so basically you just use scripting when you want and use the tag libraries like you would from Jelly, but get this, it doesn't suck!

I wrote a little utility to convert Jelly views to Groovy views, because when I found out I could convert the views in the [email-ext][5] plugin to Groovy from Jelly, I wanted to do it immediately, but who wants to convert by hand! We're software developers, we don't do things by hand. We'll spend twice as long to write a tool as it would take to do it by hand, but then, by George, if we have to do it again, it will take milliseconds!

So, the tool is strangely called [jelly2groovy][6] because you have use that naming format when writing a tool like this, just in case you didn't know that. I was converting the views in the email-ext plugin from Jelly to Groovy and the following code was generated.

<pre class="brush: groovy; title: ; notranslate" title="">f.entry(title:_("Default Subject"), help: "/plugin/email-ext/help/projectConfig/defaultSubject.html") {
  if(instance.configured) {
    input(name: "project_default_subject", value: instance.defaultSubject, class: "setting-input", type: "text")
  } 
  else {
    input(name: "project_default_subject", value: "$DEFAULT_SUBJECT", class: "setting-input", type: "text")
  }
}
}
</pre>

The thing to notice in that code is the double curly brace at the end. I only have one place that outputs a closing curly brace in the conversion script.

<pre class="brush: groovy; title: ; notranslate" title="">if( doOutput && (elem.children().size() &gt; 0) ) {
  out.writeLine("${'  ' * indent}}")
}
</pre>

I set doOutput to either true or false depending on if the tag I am rendering needs it or not (the Jelly choose tag doesn't need to be rendered, just the when/otherwise children).

So, somehow, doOutput was getting set to true, even though I set it to false inside the check for the 'choose' tag element. 

Wha?!

The code is basically like this:

<pre class="brush: groovy; title: ; notranslate" title="">doOutput = true
...
if(tag == 'choose') {
  doOutput = false
} 
...
// iterate over children by calling the current method recursively
if(doOutput...) {
   // generate the closing curly brace
}
</pre>

Not very complex. It turns out though that there is a subtle issue with the way I wrote the code and it all lies in a three letter keyword 'def'

You can read a full description of the [meaning of 'def'][7] if you would like to do so, but it boils down the following: NOT putting def in front of variable definitions in Groovy is *almost* like if the variable were global, by putting 'def' in front of the variable declaration, it refines the scope of the variable to be local. Without the 'def' in front of doOutput, when I called the method recursively and the value of doOutput was set to true, it retained that value once it got back from the recursive call and so, the ending curly brace was rendered. 

Once I figured that out, I added 'def' in front of some key variables, and things worked perfectly.

jelly2groovy is now working on several tags and does a good job of converting things over, obviously there are still tags I don't handle and things I haven't tried yet (taglibs!) but its coming along nicely.



 [1]: http://en.wikipedia.org/wiki/Open_source "Open Source"
 [2]: http://jenkins-ci.org/
 [3]: http://commons.apache.org/jelly/
 [4]: http://groovy.codehaus.org/
 [5]: https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin
 [6]: https://github.com/slide/jelly2groovy
 [7]: http://groovy.codehaus.org/Scoping+and+the+Semantics+of+%22def%22