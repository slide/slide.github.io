---
title: Prototyping and Testing Groovy Email Templates
author: alex
layout: post
permalink: /2013/02/prototyping-and-testing-groovy-email-templates/
categories:
  - groovy
  - Jenkins
---
One of the features people have requested for the <a href="https://jenkins-ci.org" title="Jenkins" target="_blank">Jenkins</a> <a href="https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin" title="email-ext" target="_blank">email-ext</a> plugin is an easier way to test out their templates for <a href="http://groovy.codehaus.org/" title="Groovy" target="_blank">Groovy</a> generated email content. (See <a href="https://issues.jenkins-ci.org/browse/JENKINS-9594" title="JENKINS-9594" target="_blank">JENKINS-9594</a>). The email-ext plugin supports Groovy&#8217;s <a href="http://groovy.codehaus.org/api/groovy/text/SimpleTemplateEngine.html" title="SimpleTemplateEngine" target="_blank">SimpleTemplateEngine</a> for generating email body (and other areas). I haven&#8217;t had the change to implement this feature yet, but found a fairly easy way to test out templates for builds based on a previous build. This can be used in the Jenkins <a href="https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+Script+Console" title="Script Console" target="_blank">Script Console</a> to test the templates on jobs. The Groovy code below will get a project that you specify by name, create a copy of it and then perform the build step for the ExtendedEmailPublisher with the previous build you want to test with. It even prints out the build log output from the ExtendedEmailPublisher running. You can change anything about the ExtendedEmailPublisher before calling perform that you might need to. This is not the final solution, I still plan on implementing this feature when I get the time to look into it more.

<pre class="brush: groovy; title: ; notranslate" title="">import hudson.model.StreamBuildListener
import hudson.plugins.emailext.ExtendedEmailPublisher
import java.io.ByteArrayOutputStream
  
def projectName = "SomeProject"
Jenkins.instance.copy(Jenkins.instance.getItem(projectName), "$projectName-Testing"); 
    
def project = Jenkins.instance.getItem(projectName)
try {
  def testing = Jenkins.instance.getItem("$projectName-Testing")
  def build = project.lastBuild
  // or def build = project.lastFailedBuild
  // see the &lt;a href="http://javadoc.jenkins-ci.org/hudson/model/Job.html#getLastBuild()" title="Job" target="_blank"&gt;javadoc for the Job class&lt;/a&gt; 
  //for other ways to get builds

  def baos = new ByteArrayOutputStream()
  def listener = new StreamBuildListener(baos)

  testing.publishersList.each() { p -&gt;
    println(p)
    if(p instanceof ExtendedEmailPublisher) {
      // modify the properties as necessary here
      p.recipientList = 'me@me.com' // set the recipient list while testing
      
      // run the publisher
      p.perform((AbstractBuild&lt;?,?&gt;)build, null, listener)
      // print out the build log from ExtendedEmailPublisher
      println(new String( baos.toByteArray(), "UTF-8" ))
    }
  }
} finally {
  if(testing != null) {
    // cleanup the test job
    testing.delete()
  }
}
</pre>

Update: Thanks to Josh Unger for a a few updates to the above to make it more robust

<span class='st\_facebook\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='Prototyping and Testing Groovy Email Templates' st\_url='http://earl-of-code.com/2013/02/prototyping-and-testing-groovy-email-templates/' displayText='pinterest'></span>