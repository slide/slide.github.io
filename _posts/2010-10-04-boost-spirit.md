---
title: Boost.Spirit
author: alex
layout: post
permalink: /2010/10/boost-spirit/
blogger_blog:
  - slide-o-blog.blogspot.com
blogger_author:
  - Alex Earlhttp://www.blogger.com/profile/09111492254896423873noreply@blogger.com
blogger_permalink:
  - /2010/10/boostspirit.html
tweet_this_url:
  - http://is.gd/pJ9qe6
categories:
  - boost spirit c++
---
I've been looking at replacing an application at work that was original written in C. The application has it's own little language for defining hardware register sets and also takes care of managing slight differences between register sets for different products. I would normally write this application using C#, but it needs to be able to run &nbsp;on systems that may or may not have .NET, and it is not an installed utility, but something that is checked into our source control system.

I thought originally about just updating the application to be C++ because it really is a good candidate for inheritance and polymorphism for the different register types, and having the STL is always a nice bonus for string manipulation. So, I need to write a parser. I've looked at Boost previously for other applications, but never at the Spirit library.

The library sounds really nice in theory, but I the examples for the latest version don't seem to follow the need that I have, or I am not understanding the examples very well.

Has anyone used the Qi library for writing a complete parser for a real life language? I'd like to see something like that.

