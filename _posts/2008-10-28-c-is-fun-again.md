---
title: C++ is fun again!
author: alex
layout: post
permalink: /2008/10/c-is-fun-again/
blogger_blog:
  - slide-o-blog.blogspot.com
blogger_author:
  - Alex Earlhttp://www.blogger.com/profile/09111492254896423873noreply@blogger.com
blogger_permalink:
  - /2008/10/c-is-fun-again.html
tweet_this_url:
  - http://is.gd/l8qhRw
categories:
  - Uncategorized
---
Lately at work I&#8217;ve been playing around with C++ and the [Boost][1] libraries; specifically I have been using [Boost.Any][2], [Boost.Filesystem][3], [Boost.Program_ptions][4], [Boost.Python][5], and last but not least [Boost smart pointers][6].

These libraries, in conjunction with some of my own macros and setup have made for a very nice transition back to C++ from C#. The awesome things about C# are the class libraries and the syntactic sugar (foreach, lambdas, anonymous methods). Similar things can be done with C++ and Boost (Boost.Foreach, etc). 

I&#8217;m going to talk about the way I&#8217;ve setup my command line options parsing, to give you an idea as to how I am using Boost.

First off, I have a static class called OptionsParser which looks like this:

<pre class="c++" name="code">class OptionsParser <br />{<br />public:<br />  static bool isParsed(void);<br />  static void parseOptions(const std::vector&lt;std::string&gt;& options);<br />  static void registerOptions(const std::string& owner, const boost::program_options::options_description& options);<br />  static void printAllOptions(std::ostream& output);<br />  static const std::vector&lt;std::string&gt; getUnrecognizedOptions(void) const;<br /><br />  template&lt;class T&gt;<br />  static T getValue(const std::string& option, T def)<br />  {<br />    T result = def;<br />    if(!isParsed())<br />      throw std::exception("You must call parseOptions before trying to retrieve values");<br /><br />    if(_variables_map.count(option))<br />      result = _variables_map[option].as&lt;T&gt;();<br /><br />    return result;<br />  }<br /><br />  template&lt;class T&gt;<br />  static T getValue(const std::string& option)<br />  {<br />    T result;<br />    if(!isParsed())<br />      throw std::exception("You must call parseOptions before trying to retrieve values");<br /><br />    if(_variables_map.count(option))<br />      result = _variables_map[option].as&lt;T&gt;();<br /><br />    return result;<br />  }<br /><br />private:<br />  OptionsParser();<br /><br />  static boost::program_options::variables_map _variables_map;<br />  static bool _parsed;<br />  static std::vector&lt;std::string&gt; _unrecognized;<br />  static boost::program_options::options_description* _all_options;<br />};<br /></pre>

The real meat of this class are the two methods registerOptions and parseOptions.

registerOptions is used from the constructor of some static class instances I have which setup the options when statics are initialized (before main).

The OptionsProvider class is what I use for this, along with some macros:

<pre class="c++" name="code">class OptionDefinition<br />{<br />public:<br />  OptionDefinition(const std::string& option_string, boost::program_options::value_semantic* value, const std::string& description);<br /><br />  const std::string& getOptionString(void) const;<br />  const boost::program_options::value_semantic* getValueSemantic(void) const;<br />  const std::string& getDescription(void) const;<br />private:<br />  std::string _option_string;<br />  boost::program_options::value_semantic* _value_semantic;<br />  std::string _description;<br />};<br /><br />class OptionsProvider<br />{<br />public:<br />  OptionsProvider(const std::string& owner, const std::string& description, OptionDefinition* options[]);<br />  virtual ~OptionsProvider(void);<br /><br />  const std::string& getOwner(void) const;<br />  const std::string& getDescription(void) const;<br />  const boost::program_options::options_description& getOptions(void) const;<br /><br />private:<br />  std::string _owner;<br />  std::string _description;<br />  boost::program_options::options_description _options;<br />};<br /><br />#define BEGIN_OPTIONS_ARRAY(owner) static OptionDefinition* Options##owner[] = { <br />#define DECLARE_OPTION(option_string, value_type, description) new OptionDefinition((option_string), \<br />new boost::program_options::typed_value&lt;value_type&gt;(NULL), \<br />(description)),<br />#define DECLARE_BOOL_OPTION(option_string, description)   new OptionDefinition((option_string), \<br />(new boost::program_options::typed_value&lt;bool&gt;(NULL))-&gt;default_value(0)-&gt;zero_tokens(), \<br />(description)),      <br /><br />#define DECLARE_DEFAULT_OPTION(option_string, value_type, def_value, description) new OptionDefinition((option_string), \<br />(new boost::program_options::typed_value&lt;value_type&gt;(NULL))-&gt;default_value(def_value), \<br />(description)),<br />#define END_OPTIONS_ARRAY()  NULL };   // add terminator to the end of the list<br /><br />#define DECLARE_OPTIONS_PROVIDER(owner, description) static OptionsProvider OptionsProvider##owner(#owner, description, Options##owner);<br /><br />#define OPTIONS_PROVIDER(owner) OptionsProvider##owner<br /></pre>

Now, here is an example usage of these macros:

<pre class="c++" name="code">BEGIN_OPTIONS_ARRAY(TestManager)<br />DECLARE_DEFAULT_OPTION("testid", unsigned long, 0L, "Step ID from automation")<br />DECLARE_DEFAULT_OPTION("timeout", int, -1, "Number of seconds before stopping a\ntest with a timeout result")<br />DECLARE_DEFAULT_OPTION("board-path", fs::path, fs::path("boards"), "Path to directory containing board libraries")<br />DECLARE_DEFAULT_OPTION("test-config", fs::path, fs::path(""), "Path to file containing the test configuration")<br />DECLARE_DEFAULT_OPTION("manager-path", fs::path, fs::path("managers"), "Path to directory container manager libraries")<br />DECLARE_DEFAULT_OPTION("output-dir", fs::path, fs::path("output"), "Path to output directory")<br />END_OPTIONS_ARRAY()<br /><br />DECLARE_OPTIONS_PROVIDER(TestManager, "TestManager options")<br /></pre>

This sets up the array and then creates a static OptionsProvider instance which will register the options with the OptionParser static class. Then I can just call OptionsParser::parseOptions(vector\_created\_from\_argv) and it will parse all the options into the boost::program\_options::variables_map.

If I have a plug-in class that is loaded later, I just do something like this after it&#8217;s options have been registered:

<pre class="c++" name="code">OptionsParser::parseOptions(OptionsParser::getUnrecognizedOptions())<br /></pre>

I could probably just have an overload of parseOptions with no parameter and have it automatically use the unrecognized options, but I haven&#8217;t decided on that yet. 

Then in my classes I can do stuff like this to get option values:

```c++
_testid = OptionsParser::getValue&lt;int&gt;("test-id");
```

to retrieve the value. If I used DECLARE\_DEFAULT\_OPTION, it automatically sets up the default value since the boost library sets that up. If I don&#8217;t, then I&#8217;d probably call the getValue overload that takes a default value.

 [1]: http://www.boost.org/
 [2]: http://www.boost.org/doc/libs/1_36_0/doc/html/any.html
 [3]: http://www.boost.org/doc/libs/1_36_0/libs/filesystem/doc/index.htm
 [4]: http://www.boost.org/doc/libs/1_36_0/doc/html/program_options.html
 [5]: http://www.boost.org/doc/libs/1_36_0/libs/python/doc/index.html
 [6]: http://www.boost.org/doc/libs/1_36_0/libs/smart_ptr/smart_ptr.htm
