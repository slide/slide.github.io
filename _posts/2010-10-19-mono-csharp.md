---
title: Mono.CSharp
author: alex
layout: post
permalink: /2010/10/mono-csharp/
blogger_blog:
  - slide-o-blog.blogspot.com
blogger_author:
  - Alex Earlhttp://www.blogger.com/profile/09111492254896423873noreply@blogger.com
blogger_permalink:
  - /2010/10/monocsharp.html
tweet_this_url:
  - http://is.gd/pDyfAe
categories:
  - 'mono c#'
---
Back in April, I read&nbsp;[this][1]&nbsp;post by Miguel de Icaza about the C# REPL (read-eval-print-loop) feature coming to MS.NET framework (previously it had only run on the Mono framework, for reasons detailed in the blog post). I was pretty excited. The feature looked really awesome when Miguel first blogged about it back in 2008 and I was pretty bummed that it was only available for Mono.&nbsp; 

<div>
</div>

<div>
  I hadn&#8217;t had an opportunity to play with it as I had been pretty busy doing other stuff at work and really didn&#8217;t want to touch the application that I wanted to add it to because it&#8217;s pretty touchy for some reason. It&#8217;s a very multithreaded application with networking, database and a bunch of other stuff, so trying to touch it can cause rippling effects. It&#8217;s something I&#8217;ve really wanted to rewrite for a while anyway. I finally decided to implement a couple new features in the application and brave the problems that would come.
</div>

<div>
</div>

<div>
  I implemented the features and it seemed like everything was working fine until the day before I went on vacation. Everything went to pot. Right down the drain. I patched up the app as best I could before I left and received some frantic pages from a colleague before I actually hit the road.
</div>

<div>
</div>

<div>
  Once I got back from my vacation I set about to correct the application correctly. You know, actually implement mutual exclusion and so forth so that it wouldn&#8217;t die a horrible death every couple days. I switched some of the threadpool stuff over to the new TPL (Task Parallel) that was released in .NET 4.0 and that has had a great improvement in speed, but I still wanted a way to break in and debug stuff at runtime. Enter the Mono.CSharp library.
</div>

<div>
</div>

<div>
  I wrote a very simple network interface that received commands from a raw connection, would evaluate them and then print back the results to the client machine. I still have a few kinks to work out, but all-in-all the solution is AWESOME. I can now remotely login and run commands to see what is happening internally in the application. I used some of the code from the example csharp.exe that Miguel released in order to have some pretty-printing and other similar features, but nothing real intense.
</div>

<div>
</div>

<div>
  If you haven&#8217;t checked out the Mono.Csharp library, I highly recommend you do. It has some potential to be very powerful. MS has promised a similar feature for the next revision of C#, but you can have it now, and very easily by just using the Mono.CSharp library.
</div>

<div>
</div>

<div>
  If you&#8217;d like more information about Mono, check out http://go-mono.com.
</div>

<span class='st\_facebook\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='Mono.CSharp' st\_url='http://earl-of-code.com/2010/10/mono-csharp/' displayText='pinterest'></span>

 [1]: http://tirania.org/blog/archive/2010/Apr-27.html