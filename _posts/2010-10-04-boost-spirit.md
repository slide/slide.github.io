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
I&#8217;ve been looking at replacing an application at work that was original written in C. The application has it&#8217;s own little language for defining hardware register sets and also takes care of managing slight differences between register sets for different products. I would normally write this application using C#, but it needs to be able to run &nbsp;on systems that may or may not have .NET, and it is not an installed utility, but something that is checked into our source control system.

I thought originally about just updating the application to be C++ because it really is a good candidate for inheritance and polymorphism for the different register types, and having the STL is always a nice bonus for string manipulation. So, I need to write a parser. I&#8217;ve looked at Boost previously for other applications, but never at the Spirit library.

The library sounds really nice in theory, but I the examples for the latest version don&#8217;t seem to follow the need that I have, or I am not understanding the examples very well.

Has anyone used the Qi library for writing a complete parser for a real life language? I&#8217;d like to see something like that.

<span class='st\_facebook\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='Boost.Spirit' st\_url='http://earl-of-code.com/2010/10/boost-spirit/' displayText='pinterest'></span>