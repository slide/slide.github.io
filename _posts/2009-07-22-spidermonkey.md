---
title: SpiderMonkey
author: alex
layout: post
permalink: /2009/07/spidermonkey/
blogger_blog:
  - slide-o-blog.blogspot.com
blogger_author:
  - Alex Earlhttp://www.blogger.com/profile/09111492254896423873noreply@blogger.com
blogger_permalink:
  - /2009/07/spidermonkey.html
tweet_this_url:
  - http://is.gd/n3kHl4
categories:
  - Uncategorized
---
The other day at work, I had a great idea come to me. For the longest time the hardware team at work has been wanting some way that they could write simple debug tests for our test boards, but they didn&#8217;t want to keep a whole source tree with everything up to date, so I thought to myself &#8220;wouldn&#8217;t it be great if I could write a simple little script parser for them?&#8221; Their needs are not great, generally just reading/writing registers on the processor and board, maybe some sort of logging and stuff like that; pretty simple really.

I started thinking I should brush up on my flex/yacc/bison/etc when it occurred to me that there had to be a simple script parser out there already that I could build into my embedded, bare metal, environment. So, I opened up google and typed in &#8220;C javascript engine&#8221; to see what I could find.

Lo, and behold, one of the first responses was for SpiderMonkey, the JS engine that is used by Firefox. Now, when I saw that, I thought there was no way that SpiderMonkey could possibly work for what I wanted, but I decided to take a look at it just to see what it looked like.

I had previously looked at the v8 scripting engine for another project and so I was expecting a rather large source tree, but SpiderMonkey 1.7 was only 1MB tar gzipped. So, I spent about 30 minutes hacking up the makefile a little bit so it would work for cross-compilation, got onto #jsapi on irc.mozilla.org and asked a couple questions about the custom implementation of dtoa and then got my first static library compiled.

Now, anyone who has ever ported something knows that compilation is a pretty big step, but just having something compiled doesn&#8217;t mean that it will work, so I didn&#8217;t have much hope.

I grabbed some simple embedding howto code that evaluated the JS code &#8220;22/7&#8243; to approximate PI and fired up my trusty compiler. Again, I had little to no hope that it would actually work, but to my surprise I got a nice print-out on my screen of the approximation of PI.

I spent another 30 minutes implementing some wrappers for some native functions (logging, etc) for JS and wrote some simple little test scripts.

I was very impressed by SpiderMonkey 1.7, it was easy to hack up a little to get it to cross-compile, it doesn&#8217;t require a lot of external libraries (none really) and it runs very nice on my embedded target.

My next step will be to try with one of the releases that includes TraceMonkey and the speed improvements that come with JIT.

I have yet to show my idea to the hardware team at work, but I think they will like it. I will support a simple executable that takes a script on the command line and executes it. They just need to write the simple test vectors they need and don&#8217;t have to worry about the whole code environment.

Thanks SpiderMonkey!

<span class='st\_facebook\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='SpiderMonkey' st\_url='http://earl-of-code.com/2009/07/spidermonkey/' displayText='pinterest'></span>