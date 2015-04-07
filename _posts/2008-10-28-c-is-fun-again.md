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
Lately at work I've been playing around with C++ and the [Boost][1] libraries; specifically I have been using [Boost.Any][2], [Boost.Filesystem][3], [Boost.ProgramOptions][4], [Boost.Python][5], and last but not least [Boost smart pointers][6].

These libraries, in conjunction with some of my own macros and setup have made for a very nice transition back to C++ from C#. The awesome things about C# are the class libraries and the syntactic sugar (foreach, lambdas, anonymous methods). Similar things can be done with C++ and Boost (Boost.Foreach, etc). 

I'm going to talk about the way I've setup my command line options parsing, to give you an idea as to how I am using Boost.

First off, I have a static class called OptionsParser which looks like this:

{% highlight cpp linenos=table %}
class OptionsParser 
{
public:
  static bool isParsed(void);
  static void parseOptions(const std::vector<std::string>& options);
  static void registerOptions(const std::string& owner, const boost::program_options::options_description& options);
  static void printAllOptions(std::ostream& output);
  static const std::vector<std::string> getUnrecognizedOptions(void) const;

  template<class T>
  static T getValue(const std::string& option, T def)
  {
    T result = def;
    if(!isParsed())
      throw std::exception("You must call parseOptions before trying to retrieve values");
    if(_variables_map.count(option))
      result = _variables_map[option].as<T>();

    return result;
  }

  template<class T>
  static T getValue(const std::string& option)
  {
    T result;
    if(!isParsed())
      throw std::exception("You must call parseOptions before trying to retrieve values");

    if(_variables_map.count(option))
      result = _variables_map[option].as<T>();
    return result;
  }

private:
  OptionsParser();
  static boost::program_options::variables_map _variables_map;
  static bool _parsed;
  static std::vector<std::string> _unrecognized;
  static boost::program_options::options_description* _all_options;
}
{% endhighlight %}

The real meat of this class are the two methods registerOptions and parseOptions.

registerOptions is used from the constructor of some static class instances I have which setup the options when statics are initialized (before main).

The OptionsProvider class is what I use for this, along with some macros:

{% highlight cpp linenos=table %}
class OptionDefinition
{
public:
  OptionDefinition(const std::string& option_string, boost::program_options::value_semantic* value, const std::string& description);
  const std::string& getOptionString(void) const;
  const boost::program_options::value_semantic* getValueSemantic(void) const;
  const std::string& getDescription(void) const;
private:
  std::string _option_string;
  boost::program_options::value_semantic* _value_semantic;
  std::string _description;
};

class OptionsProvider
{
public:
  OptionsProvider(const std::string& owner, const std::string& description, OptionDefinition* options[]);
  virtual ~OptionsProvider(void);

  const std::string& getOwner(void) const;
  const std::string& getDescription(void) const;
  const boost::program_options::options_description& getOptions(void) const;

private:
  std::string _owner;
  std::string _description;
  boost::program_options::options_description _options;
};

#define BEGIN_OPTIONS_ARRAY(owner) static OptionDefinition* Options##owner[] = {
#define DECLARE_OPTION(option_string, value_type, description) new OptionDefinition((option_string), \
  new boost::program_options::typed_value<value_type>(NULL), \
  (description)),
#define DECLARE_BOOL_OPTION(option_string, description)   new OptionDefinition((option_string), \
  (new boost::program_options::typed_value<bool>(NULL))->default_value(0)->zero_tokens(), \
  (description)),
#define DECLARE_DEFAULT_OPTION(option_string, value_type, def_value, description) new OptionDefinition((option_string), \
  (new boost::program_options::typed_value<value_type>(NULL))->default_value(def_value), \
  (description)),
#define END_OPTIONS_ARRAY()  NULL };   // add terminator to the end of the list

#define DECLARE_OPTIONS_PROVIDER(owner, description) static OptionsProvider OptionsProvider##owner(#owner, description, Options##owner);
#define OPTIONS_PROVIDER(owner) OptionsProvider##owner
{% endhighlight %}

Now, here is an example usage of these macros:

{% highlight cpp linenos=table %}
BEGIN_OPTIONS_ARRAY(TestManager)
  DECLARE_DEFAULT_OPTION("testid", unsigned long, 0L, "Step ID from automation")
  DECLARE_DEFAULT_OPTION("timeout", int, -1, "Number of seconds before stopping a\ntest with a timeout result")
  DECLARE_DEFAULT_OPTION("board-path", fs::path, fs::path("boards"), "Path to directory containing board libraries")
  DECLARE_DEFAULT_OPTION("test-config", fs::path, fs::path(""), "Path to file containing the test configuration")
  DECLARE_DEFAULT_OPTION("manager-path", fs::path, fs::path("managers"), "Path to directory container manager libraries")
  DECLARE_DEFAULT_OPTION("output-dir", fs::path, fs::path("output"), "Path to output directory")
END_OPTIONS_ARRAY()

DECLARE_OPTIONS_PROVIDER(TestManager, "TestManager options")
{% endhighlight %}

This sets up the array and then creates a static OptionsProvider instance which will register the options with the OptionParser static class. Then I can just call OptionsParser::parseOptions(vector\_created\_from\_argv) and it will parse all the options into the boost::program\_options::variables_map.

If I have a plug-in class that is loaded later, I just do something like this after it&#8217;s options have been registered:

{% highlight cpp linenos=table %}
OptionsParser::parseOptions(OptionsParser::getUnrecognizedOptions())
{% endhighlight %}

I could probably just have an overload of parseOptions with no parameter and have it automatically use the unrecognized options, but I haven&#8217;t decided on that yet. 

Then in my classes I can do stuff like this to get option values:

{% highlight cpp linenos=table %}
_testid = OptionsParser::getValue<int>("test-id");
{% endhighlight %}

to retrieve the value. If I used DECLARE\_DEFAULT\_OPTION, it automatically sets up the default value since the boost library sets that up. If I don&#8217;t, then I&#8217;d probably call the getValue overload that takes a default value.

 [1]: http://www.boost.org/
 [2]: http://www.boost.org/doc/libs/1_36_0/doc/html/any.html
 [3]: http://www.boost.org/doc/libs/1_36_0/libs/filesystem/doc/index.htm
 [4]: http://www.boost.org/doc/libs/1_36_0/doc/html/program_options.html
 [5]: http://www.boost.org/doc/libs/1_36_0/libs/python/doc/index.html
 [6]: http://www.boost.org/doc`/libs/1_36_0/libs/smart_ptr/smart_ptr.htm
