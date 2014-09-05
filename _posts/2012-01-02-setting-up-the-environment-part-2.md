---
title: 'Setting up the environment &ndash; Part 2'
author: alex
layout: post
permalink: /2012/01/setting-up-the-environment-part-2/
tweet_this_url:
  - http://is.gd/67s0Uw
categories:
  - 'C#'
  - IronPython
  - 'mono c#'
---
<p align="left">
  Part 1 of this series is available <a href="http://earl-of-code.com/2011/12/ironpython-and-pyexpat-part-1/">here</a>
</p>

<p align="left">
  To begin porting a C Python module to IronPython a few things are needed, I will explain how to get some of them and leave the rest up to the reader to research and download/install. I am assuming that you will be doing development on Windows, if you prefer Linux, there are equivalents available and I will note them below.
</p>

  * <div align="left">
      .NET Framework 4.0 SDK (Windows) or Mono 2.10.x (Linux)
    </div>
    
      * <div align="left">
          This includes tools such as msbuild (xbuild for Mono), the C# compiler and other such tools. This is the bare minimum required for building IronPython from sources.
        </div>
    
      * <div align="left">
          TortoiseGit (or any other Git client) for Windows or git (Linux)
        </div>
        
          * <div align="left">
              This will be used to get the latest sources from Github for IronPython.
            </div>
        
          * <div align="left">
              Visual Studio 2010 (Windows recommended) or MonoDevelop 2.8 (Linux)
            </div>
            
              * <div align="left">
                  You CAN build IronPython sources using only the .NET SDK, but having a good debugger makes life much easier.
                </div>
            
              * <div align="left">
                  Python 2.7.2 sources (<a href="http://www.python.org">http://www.python.org</a>)
                </div>
                
                  * <div align="left">
                      The Python sources are required so that you can look at internal implementations of C based modules to make sure your IronPython modules match the implementation.
                    </div></ul> 
            
            <p align="left">
              If you are developing on Linux, I highly recommend either getting the latest Mono tarball, or building from source yourself. The latest Mono in the various Linux distributions’ repositories are not the latest and greatest release, and some of that latest and greatest stuff is nice to have when developing for IronPython.
            </p>
            
            <p align="left">
              If you are not familiar with open source development on Github, here’s a quick and dirty tutorial which should get you going.
            </p>
            
            <p align="left">
              Github is called a social development platform. The idea is to get developers communicating back and forth and provide ways for communities to work together. That being said, the major benefit of Github for open source projects is that is makes contributing very nice and much less painful than other methods. The general way to contribute to a project is to fork it. To fork a project on Github means that you create your own working area for the project, with it’s own source control history of your changes, and changes that you incorporate from other people into your branch of the source.
            </p>
            
            <p align="left">
              Obviously, working on Github requires an account, so if you don’t already have one, you’ll need to sign up for an account, then you can fork to your heart’s content. I would also like to mention that strictly speaking, you do not have to setup a Github account, if you have access to your own git repository and server, you could theoretically send patches to the IronPython team that way, but it is MUCH more preferred to work using the Github method.
            </p>
            
            <p align="left">
              When you browse to the IronPython project on Github (which is actually part of the IronLanguages project) at <a href="https://github.com/ironlanguages/main">https://github.com/ironlanguages/main</a> you’ll see something like below (depending on if you are logged in or not).
            </p>
            
            <p align="left">
              <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb.png" width="569" height="482" /></a>
            </p>
            
            <p align="left">
              Github provides you several ways of getting the source for the project.
            </p>
            
              1. <div align="left">
                  Click the “ZIP” link right above the “Files” tab of the UI. This will grab the head (latest) of the source code and allow you to download it as a zip file. If you want to peruse the source and see how things are done before jumping in head first, this can be a good way of doing that.
                </div>
                
                  * <div align="left">
                      Depending on your role in a project (developer, administrator, etc.) you will have the option of having a direct committable URL for the project. There are SSH and HTTP options available for developers and administrators that allow directly pushing changes to the main project. Most people do NOT have this access.
                    </div>
                    
                      * <div align="left">
                          The other option, for those who are not in the project developers group directly, but would still like to get the source with all the Git information is the “Git Read-Only” option. This allows you to git [sic] the source and pull updates, but does not allow you to commit anything to the project.
                        </div></ol> 
                    
                    <p align="left">
                      These options are for if you want to get the source only and not really hack on it. If you want to hack on the source and contribute back new features or bug fixes, then you will want to fork the project<sup><a href="#references1">1</a></sup>.
                    </p>
                    
                    <p align="left">
                      <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image1.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb1.png" width="569" height="60" /></a>
                    </p>
                    
                    <p align="left">
                      <p>
                        The important button in this case is the “Fork” button which is near the top of the page. As shown above, you have to be logged in to be able to fork a project. Forking a project creates your own personal work area. You are automatically added to the “developers” list for the project and given push access to your fork of the project. Github will tell you there is hardcore forking action going on and then redirect you to the project page for your fork as shown below.
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image2.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb2.png" width="569" height="280" /></a>
                      </p>
                      
                      <p>
                        You can see that the project in my case is now “slide / ironlanguages” not “IronLanguages / main.” When you fork IronPython specifically, you will actually end up with a project called “username / main” which in my mind was not very descriptive of what it actually was, but luckily Github lets you rename your fork. Click on the “Admin” button on the fork’s project page, you can change the name in the repository settings.
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image3.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb3.png" width="488" height="157" /></a>
                      </p>
                      
                      <p>
                        I renamed mine to ironlanguages, so I knew exactly what it was (I contribute to some other open source projects on Github, so keeping them straight is important).
                      </p>
                      
                      <p>
                        Now that you have a fork, you are ready to get the sources and start hacking away. Browse to the directory on your computer where you would like to keep the source and use the Git client to clone the repository URL displayed on your fork<sup><a href="#references2">2</a></sup>.
                      </p>
                      
                      <p>
                        If you are using TortoiseGit on Windows, browse to the root of where you want the source and right click to show the TortoiseGit context menu entries.
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image4.png"><img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb4.png" width="244" height="63" /></a>
                      </p>
                      
                      <p>
                        Select “Git Clone…”
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image5.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb5.png" width="442" height="335" /></a>
                      </p>
                      
                      <p>
                        Most of the necessary information will be filled in for you when you paste in the URL for the repository you want to clone, if not use the image above to help.
                      </p>
                      
                      <p>
                        You will notice that I am loading a Putty key, this helps with authorization and not having to enter a password for every operation with Github. I highly recommend you follow the how to on Github for setting up your keys<sup><a href="#references3">3</a></sup>.
                      </p>
                      
                      <p>
                        When you press OK, a progress dialog will appear showing you the progress of the clone.
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image6.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb6.png" width="569" height="449" /></a>
                      </p>
                      
                      <p>
                        This will take some time as there is quite a bit of code in the repository (including IronRuby, the DLR, Python files for the standard library, and more). If everything goes as planned, you will see something like the following:
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image7.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb7.png" width="560" height="442" /></a>
                      </p>
                      
                      <p>
                        Now, you should have a local copy of the repo in the directory you specified.
                      </p>
                      
                      <p>
                        <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image8.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb8.png" width="560" height="380" /></a>
                      </p>
                      
                      <p>
                        Interesting areas to look at in the code.
                      </p>
                      
                      <ol>
                        <li>
                          Solutions directory &#8211; contains the Visual Studio solution files for the various projects that can be built (remember to use IronPython.Mono.sln if building on Linux). <li>
                            Languages\IronPython\IronPython.Modules – contains most of the “native” modules that are ports from C modules <li>
                              Languages\IronPython\IronPython – contains the actual implementation (parser, compiler, etc.) for IronPython
                            </li></ol> 
                            <p>
                              Once you have the IronPython sources on your local system, take a look at the C Python source code.
                            </p>
                            
                            <p>
                              <a href="http://earl-of-code.com/wp-content/uploads/2012/01/image9.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://earl-of-code.com/wp-content/uploads/2012/01/image_thumb9.png" width="569" height="385" /></a>
                            </p>
                            
                            <p>
                              Interesting areas to look at.
                            </p>
                            
                            <ol>
                              <li>
                                Modules – contains the implementation of the “native” modules (written in C), this is where we look for the C Python implementations for modules we want to port. <li>
                                  Python – contains implementations of several key Python components such as the importer, dynamic loading, and more. This can be useful to determine what Python API function calls in the C source code are doing so the functionality can be replicated.
                                </li></ol> 
                                <p>
                                  Next time, I’ll start pulling apart the C Python pyexpat module source code and create a skeleton IronPython module.
                                </p>
                                
                                <p>
                                  References
                                </p>
                                
                                <ol>
                                  <li>
                                    <a name="references1"></a>Github has a much better tutorial on forking a project available at <a href="http://help.github.com/fork-a-repo/">http://help.github.com/fork-a-repo/</a> <li>
                                      <a name="references2"></a>Github’s documentation <a href="http://help.github.com/">http://help.github.com/</a> is pretty good, I look through it all the time. <li>
                                        <a name="references3"></a>Look at <a href="http://nathanj.github.com/gitguide/tour.html">http://nathanj.github.com/gitguide/tour.html</a> under &#8220;Pushing to a Remote Server&#8221; for information on generating your own SSH keys for use with Github.
                                      </li></ol> 
                                      <p>
                                        <span class='st_facebook_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='facebook'></span><span class='st_twitter_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='twitter'></span><span class='st_linkedin_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='linkedin'></span><span class='st_email_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='email'></span><span class='st_sharethis_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='sharethis'></span><span class='st_fblike_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='fblike'></span><span class='st_plusone_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='plusone'></span><span class='st_pinterest_vcount' st_title='Setting up the environment &ndash; Part 2' st_url='http://earl-of-code.com/2012/01/setting-up-the-environment-part-2/' displayText='pinterest'></span>
                                      </p>