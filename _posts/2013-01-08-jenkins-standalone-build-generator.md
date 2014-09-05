---
title: 'Jenkins &#8211; Standalone Build Generator'
author: alex
layout: post
permalink: /2013/01/jenkins-standalone-build-generator/
categories:
  - Uncategorized
---
I&#8217;ve blogged before about how we use Jenkins at work for our continuous integration solution. One thing that our previous CI solution had was the ability for developers to run a standalone version of the tool on their development PC&#8217;s to check out large scale changes that might break several applications. Jenkins is much more difficult to do this with, mainly because we are using Rational Clearcase for SCM. We could use the Jenkins server to build from individual development streams if we wanted to, but it would require that all the files be checked in before being able to build locally.

I came up with a Groovy script that runs after each Nightly Build that collects the jobs that are currently in Jenkins and generates a standalone zip file that developers can download and launch a local instance of Jenkins to do a build from their view. The script is used as a  Groovy build step.

<noscript>
  <pre><code class="language-groovy groovy">import jenkins.model.*
import hudson.model.*

import java.io.*
import groovy.xml.*
  
import java.util.zip.ZipOutputStream  
import java.util.zip.ZipEntry  
import java.nio.channels.FileChannel  
import java.util.regex.Pattern

// Retrieve parameters of the current build
def build = Thread.currentThread().executable

def build_id = 'build' + build.id

def root_dir = new File(build.workspace.toString())
def job_dir = new File(root_dir, 'jobs')
job_dir.mkdirs()
  
files = []
builds = []

def parentBuild = '''&lt;?xml version='1.0' encoding='UTF-8'?&gt;
&lt;project&gt;
  &lt;actions/&gt;
  &lt;description&gt;&lt;/description&gt;
  &lt;keepDependencies&gt;false&lt;/keepDependencies&gt;
  &lt;properties/&gt;
  &lt;scm class="hudson.scm.NullSCM"/&gt;
  &lt;canRoam&gt;true&lt;/canRoam&gt;
  &lt;disabled&gt;false&lt;/disabled&gt;
  &lt;blockBuildWhenDownstreamBuilding&gt;false&lt;/blockBuildWhenDownstreamBuilding&gt;
  &lt;blockBuildWhenUpstreamBuilding&gt;false&lt;/blockBuildWhenUpstreamBuilding&gt;
  &lt;triggers class="vector"/&gt;
  &lt;concurrentBuild&gt;false&lt;/concurrentBuild&gt;
  &lt;builders/&gt;
  &lt;publishers/&gt;
  &lt;buildWrappers/&gt;
&lt;/project&gt;'''

def jenkinsConfigXml = '''&lt;?xml version='1.0' encoding='UTF-8'?&gt;
&lt;hudson&gt;
  &lt;disabledAdministrativeMonitors/&gt;
  &lt;version&gt;1.477&lt;/version&gt;
  &lt;numExecutors&gt;1&lt;/numExecutors&gt;
  &lt;mode&gt;NORMAL&lt;/mode&gt;
  &lt;useSecurity&gt;true&lt;/useSecurity&gt;
  &lt;authorizationStrategy class="hudson.security.AuthorizationStrategy$Unsecured"/&gt;
  &lt;securityRealm class="hudson.security.SecurityRealm$None"/&gt;
  &lt;projectNamingStrategy class="jenkins.model.ProjectNamingStrategy$DefaultProjectNamingStrategy"/&gt;
  &lt;workspaceDir&gt;${ITEM_ROOTDIR}/workspace&lt;/workspaceDir&gt;
  &lt;buildsDir&gt;${ITEM_ROOTDIR}/builds&lt;/buildsDir&gt;
  &lt;jdks/&gt;
  &lt;viewsTabBar class="hudson.views.DefaultViewsTabBar"/&gt;
  &lt;myViewsTabBar class="hudson.views.DefaultMyViewsTabBar"/&gt;
  &lt;clouds/&gt;
  &lt;slaves/&gt;
  &lt;quietPeriod&gt;5&lt;/quietPeriod&gt;
  &lt;scmCheckoutRetryCount&gt;0&lt;/scmCheckoutRetryCount&gt;
  &lt;views&gt;
    &lt;hudson.model.AllView&gt;
      &lt;owner class="hudson" reference="../../.."/&gt;
      &lt;name&gt;All&lt;/name&gt;
      &lt;filterExecutors&gt;false&lt;/filterExecutors&gt;
      &lt;filterQueue&gt;false&lt;/filterQueue&gt;
      &lt;properties class="hudson.model.View$PropertyList"/&gt;
    &lt;/hudson.model.AllView&gt;
  &lt;/views&gt;
  &lt;primaryView&gt;All&lt;/primaryView&gt;
  &lt;slaveAgentPort&gt;0&lt;/slaveAgentPort&gt;
  &lt;label&gt;&lt;/label&gt;
  &lt;nodeProperties/&gt;
  &lt;globalNodeProperties/&gt;
&lt;/hudson&gt;'''

def binCopyXml = '''&lt;?xml version='1.0' encoding='UTF-8'?&gt;
&lt;project&gt;
  &lt;actions/&gt;
  &lt;description&gt;&lt;/description&gt;
  &lt;keepDependencies&gt;false&lt;/keepDependencies&gt;
  &lt;properties/&gt;
  &lt;scm class="hudson.scm.NullSCM"/&gt;
  &lt;canRoam&gt;true&lt;/canRoam&gt;
  &lt;disabled&gt;false&lt;/disabled&gt;
  &lt;blockBuildWhenDownstreamBuilding&gt;false&lt;/blockBuildWhenDownstreamBuilding&gt;
  &lt;blockBuildWhenUpstreamBuilding&gt;false&lt;/blockBuildWhenUpstreamBuilding&gt;
  &lt;triggers class="vector"/&gt;
  &lt;concurrentBuild&gt;false&lt;/concurrentBuild&gt;
  &lt;builders&gt;
    &lt;hudson.tasks.Shell&gt;
      &lt;command&gt;/usr/bin/find ${VIEW_ROOT} -iname &apos;*.bin&apos;

/usr/bin/mkdir -p &quot;${RELEASE_DIR}&quot;
/usr/bin/find ${VIEW_ROOT} -iname &apos;*.bin&apos; -print0 | /usr/bin/xargs -0 /usr/bin/cp -ut &quot;${RELEASE_DIR}&quot;
&lt;/command&gt;
    &lt;/hudson.tasks.Shell&gt;
  &lt;/builders&gt;
  &lt;publishers/&gt;
  &lt;buildWrappers/&gt;
&lt;/project&gt;'''


for(it in Jenkins.instance.items) {
  if(it.name.contains('Hourly-Build') || it.name.contains('Daily-Build') || it.name.startsWith('ZZ')) continue
  
  def this_job_dir = new File(job_dir, it.name)
  this_job_dir.mkdirs()
  def project = new XmlSlurper().parseText(it.configFile.asString())
  
  // clean out stuff we don't need in the standalone build (email notification, etc)
  project.builders."hudson.tasks.BatchFile".replaceNode {}
  project.publishers."hudson.plugins.warnings.WarningsPublisher".replaceNode {}
  project.publishers."hudson.plugins.postbuildtask.PostbuildTask".replaceNode {}
  project.publishers."hudson.plugins.emailext.ExtendedEmailPublisher".replaceNode {}
  project.buildWrappers."hudson.plugins.setenv.SetEnvBuildWrapper".replaceNode {}
  project.properties."hudson.model.ParametersDefinitionProperty".replaceNode {}

  configXml = new File(this_job_dir, "config.xml")
    
  res = new StreamingMarkupBuilder().bind{
    mkp.yield project
  }
    
  def outFile = new FileWriter(configXml)
  def out = new PrintWriter(outFile)
  out.write(res)
  out.close()
  outFile.close()
  files.add(configXml)
  builds.add(it.name)
  println('Wrote: ' + configXml)
}

/*
&lt;hudson.tasks.BuildTrigger&gt;
      &lt;childProjects&gt;PROJECTS&lt;/childProjects&gt;
      &lt;threshold&gt;
        &lt;name&gt;SUCCESS&lt;/name&gt;
        &lt;ordinal&gt;0&lt;/ordinal&gt;
        &lt;color&gt;BLUE&lt;/color&gt;
      &lt;/threshold&gt;
    &lt;/hudson.tasks.BuildTrigger&gt;
*/

// creat a build to copy binaries to the RELEASE_DIR
builds.add('ZZ-Binary-Copy')

def project = new XmlSlurper().parseText(parentBuild)
project.publishers.appendNode {
  "hudson.tasks.BuildTrigger" {
    childProjects(builds.join(','))
    threshold {
       name("SUCCESS")
       ordinal("0")
       color("BLUE")
    }
  }
}


def outFile = new FileWriter(new File(root_dir, 'config.xml'))
def out = new PrintWriter(outFile)
out.write(jenkinsConfigXml)
out.close()

// create a build to launch all the other builds
def buildAllDir = new File(job_dir, '00-Build-All')
buildAllDir.mkdirs()
outFile = new FileWriter(new File(buildAllDir, 'config.xml'))
out = new PrintWriter(outFile)
out.write(new StreamingMarkupBuilder().bind {
    mkp.yield project
})
out.close()

def binCopyDir = new File(job_dir, 'ZZ-Binary-Copy')
binCopyDir.mkdirs()
outFile = new FileWriter(new File(binCopyDir, 'config.xml'))
out = new PrintWriter(outFile)
out.write(binCopyXml)
out.close()

// generate the bat file and zip file
def zipFileName = new File(root_dir, build_id + ".zip")

def root_dir_length = root_dir.absolutePath.length() + 1

def ignorePattern = Pattern.compile('\\.zip')

def jenkins_war = new File(new File(Jenkins.instance.servletContext.getRealPath(Jenkins.instance.servletContext.contextPath)).getParentFile().getParentFile(), 'jenkins.war')

def batfile = '''@echo off

set JENKINS_HOME=%cd%\\jenkins

IF NOT "%~1" == "" SET VIEW_ROOT=%1
IF NOT "%~2" == "" SET RELEASE_DIR=%2
IF "%VIEW_ROOT%." == "." GOTO ViewError
IF "%RELEASE_DIR%." == "." Goto ReleaseDirError

set PATH=C:\\cygwin\\bin;%PATH%

FOR /F "tokens=* USEBACKQ" %%A IN (`c:\\cygwin\\bin\\cygpath.exe "%VIEW_ROOT%"`) DO SET VIEW_ROOT=%%A
FOR /F "tokens=* USEBACKQ" %%A IN (`c:\\cygwin\\bin\\cygpath.exe "%RELEASE_DIR%"`) DO SET RELEASE_DIR=%%A

start "Jenkins" java -server -jar jenkins.war --httpPort=8099
echo "Jenkins will start and a browser window will be opened within 10 seconds"
@ping 127.0.0.1 -n 10 -w 1000&gt; nul
start "Jenkins Web" http://localhost:8099
GOTO Done
:ViewError
ECHO "You must set the VIEW_ROOT environment variable (or pass it as the first parameter) to the PATH of the view to build in"
GOTO Done
:ReleaseDirError
ECHO "You must set the RELEASE_DIR environment variable (or pass it as the second parameter) to the PATH you want the binaries copied to"

:Done
'''

is = new ByteArrayInputStream(batfile.getBytes());

def zipFile = new ZipOutputStream(new FileOutputStream(zipFileName)) 
zipFile.putNextEntry(new ZipEntry('jenkins.war'))
zipFile &lt;&lt; new FileInputStream(jenkins_war)
zipFile.putNextEntry(new ZipEntry('run_build_server.bat'))
zipFile &lt;&lt; is
 
root_dir.eachFileRecurse { file -&gt;
  def relative = file.absolutePath.substring(root_dir_length).replace('\\', '/') 
  if ( file.isDirectory() && !relative.endsWith('/')){
    relative += "/"
  }
  if( ignorePattern.matcher(relative).find() ){
    return
  }

  zipFile.putNextEntry(new ZipEntry('jenkins/' + relative))  
  if( file.isFile() ){
    zipFile &lt;&lt; new FileInputStream(file)
  }  
}  
zipFile.close() </code></pre>
</noscript>

<span class='st\_facebook\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='Jenkins &#8211; Standalone Build Generator' st\_url='http://earl-of-code.com/2013/01/jenkins-standalone-build-generator/' displayText='pinterest'></span>