---
title: Testing With IronPython
author: alex
layout: post
permalink: /2012/02/testing-with-ironpython/
tweet_this_url:
  - http://is.gd/38Rpeo
wp_plus_one_redirect:
  - ""
categories:
  - 'C#'
  - IronPython
---
At my work, we do validation of system on a chip devices. We have our own internal hardware team that designs boards for us to use in testing. These boards are complicated pieces of equipment, containing multiple FPGAs, power supply components, and more. With this complication comes the possibility that something could go wrong with the board. I develop an application used by other engineers to use these boards to load and run their tests (which run on the embedded device). For a long time, in this application we had some simple board tests that would help debug issues as they came up. We started getting more and more products and the boards changed slightly and it was difficult to keep these tests up to par with what needed to be there for the different products. I had been playing with IronPython for some time on various projects at home and thought this might be a good place to use it. The idea being that changes are more easily made in scripts, and the hardware team could add new tests as problems crop up.

I decided that the unittest module for Python would do a great job of managing the tests that needed to be run and allowing grouping of the tests into test suites that could be run by different users.

I’m going to take you through the design and implementation of an application similar to that which I use at work to help our hardware team debug boards. It will not be the exact application I use at work, but will give you an idea of some of the simple, yet powerful things you can do with IronPython.

[<img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" src="http://earl-of-code.com/wp-content/uploads/2012/02/image_thumb.png" alt="image" width="569" height="416" border="0" />][1]

The UI is very simple: a simple menu with a single entry (File > Exit), a toolbar with four buttons, a listview that shows the currently loaded set of tests and an output window for the test output. Here you can see I loaded the test_datetime.py file into the GUI to run the datetime tests (and sadly it looks like there are failures in IronPython’s datetime module!).

I’m not going to go into heavy detail about the GUI development, that is not the important part of this exercise. Let’s walk through the steps necessary to load in a Python file that contains unit tests.

We’ve got a couple issues to solve for this application:

  1. How do we get the output from the unit tests to go to the output text box?
  2. How do we enumerate and show all the unit tests in the given file?

First things first, we need to get IronPython going inside of our application. Download the latest stable version from <http://ironpython.codeplex.com> (as of this writing, the latest is 2.7.1 which means its roughly compatible with CPython 2.7). For embedding IronPython into an application, I prefer to get the zip file release that contains the necessary assemblies. Unzip the distribution to any place you would like and add the following as references to your project:

  1. IronPython.dll
  2. IronPython.Modules.dll
  3. Microsoft.Dynamic.dll
  4. Microsoft.Scripting.dll
  5. Microsoft.Scripting.MetaData.dll

These are the basic assemblies that will be used for embedding IronPython into the application.

The creation of the GUI as I said, is not the important part of this tutorial, if you have questions about anything I did feel free to comment, or drop me an email.

So, lets start getting our application ready to run Python code.

We need an execution engine.For this application, I am only planning on supporting one engine for the entire application, I don’t have a need to support an engine per thread or anything like that. So, I’ll go ahead and create a ScriptEngine (Microsoft.Scripting.Hosting) at the class level of my main form.

Most often when I am embedding IronPython into an application, I will have a method to initialize the engine and setup any paths I may need, pre-import some modules and do various other initialization steps. Here is the InitializeEngine method.

<noscript>
  <pre><code class="language-c# c#">void InitializeEngine() {
     // first clear out anything we have now
     if (_scope != null)
         _scope = null;

     if (_engine != null)
         _engine = null;

     _engine = Python.CreateEngine();
     _scope = _engine.CreateScope();

     _stdout = new OutputStream();
     _stderr = new OutputStream();
     _stdout.Output += stdout_Output;
     _stderr.Output += stderr_Output;

     _engine.Runtime.IO.SetOutput(_stdout, Encoding.ASCII);
     _engine.Runtime.IO.SetErrorOutput(_stderr, Encoding.ASCII);

     // add the local python installation libs
     ICollection&lt;string&gt; searchPaths = _engine.GetSearchPaths();
     searchPaths.Add(Path.Combine(PYTHON_DIR, "Lib"));
     searchPaths.Add(Path.Combine(Path.Combine(PYTHON_DIR, "Lib"), "site-packages"));
     searchPaths.Add(Path.GetDirectoryName(GetType().Assembly.Location));

     _engine.SetSearchPaths(searchPaths);

     // import some default stuff.
     Import("sys", "os", "unittest", "testermatic");
     _stdout.WriteLine("Ready...");
}</code></pre>
</noscript>

A few things of note. You can see the creation of a ScriptScope object as well as the ScriptEngine object we already talked about. This ScriptScope can be thought of as the \_\_main\_\_ module for the execution of the Python code. With IronPython you can create multiple ScriptScopes per engine and execute your code in any of them and they are partially self-contained.

The code is also creating and setting up the stdout and stderr handlers for the ScriptEngine. This was an easy way to run the unit tests and get their output (since the default test running just prints the results to the console using the Python print statement). The OutputStream implementation is fairly simple and is shown below.

<noscript>
  <pre><code class="language-c# c#">using System;
using System.IO;
using System.Text;

namespace Boardom {
    class OutputStream : MemoryStream {
        public class OutputEventArgs : EventArgs {
            public OutputEventArgs(string text) {
                Text = text;
            }

            public string Text {
                get;
                private set;
            }
        }

        public event EventHandler&lt;OutputEventArgs&gt; Output;

        public OutputStream()
            : base() {
        }

        public override void Write(byte[] buffer, int offset, int count) {
            string result = Encoding.ASCII.GetString(buffer, offset, count);
            if (!string.IsNullOrEmpty(result))
                OnOutput(result);
        }        

        public void WriteLine(string format, params object[] args) {
            string result = string.Format(format, args);
            if (!result.EndsWith("\n"))
                result = result + "\n";

            if (!string.IsNullOrEmpty(result))
                OnOutput(result);
        }

        void OnOutput(string text) {
            if (Output != null)
                Output(this, new OutputEventArgs(text));
        }
    }
}
</code></pre>
</noscript>

It uses events to send text to the main form when something is written to the stream. The InitializeEngine then sets the stdout and stderr for the ScriptEngine; now all text written to either stdout or stderr will be redirected and displayed in the output area.

Now that an engine is setup, and the stdout and stderr output is redirected, the InitializeEngine method pre-imports a few modules so that the script writer doesn’t need to do so. The first three (sys, os, and unittest) are standard Python modules that come in the Lib directory of the IronPython distribution. The last one is a helper module used to help enumerate and execute the unit tests.

<noscript>
  <pre><code class="language-python python">import clr, unittest, sys

class TestermaticTestCase(unittest.TestCase):
    def __init__(self, testcase):
        self._listeners = []
        self._testcase = testcase

    def addListener(self, listener):
        if listener:
            self._listeners.append(listener)
            
    def setUp(self):
        self._testcase.setUp()
        
    def tearDown(self):
        self._testcase.tearDown()
        
    def countTestCases(self):
        return self._testcase.countTestCases()

    def shortDescription(self):
        return self._testcase.shortDescription()
        
    def id(self):
        return self._testcase.id()
        
    def __str__(self):
        return self._testcase.__str__
        
    def __repr__(self):
        return self._testcase.__repr__
        
    def run(self, result=None):
        self._testcase.run(result)
        if result:
            for listener in self._listeners:
                listener(self, result.testsRun, result.failures, result.errors)
        return result
        


def getTests(prefix='test'):
    """ Finds all tests in all loaded modules, except for certain predefined modules """
    tests = []
    for mod_name in sys.modules:
        if not mod_name in ['unittest', 'testermatic', 'clr', 'unittest.case']:
#            print 'mod_name = %s' % mod_name
            testsuites = unittest.findTestCases(sys.modules[mod_name], prefix)
            if testsuites:
                for testsuite in testsuites:
                    for test in testsuite:
                        tests.append(test)	
    return tests
    

def runTests(test_list, complete_func=None):
    suite = unittest.TestSuite()
    for test in test_list:
        print test
        if complete_func:
            test = TestermaticTestCase(test)
            test.addListener(complete_func)			
        suite.addTest(test)	
    
    return unittest.TextTestRunner().run(suite)</code></pre>
</noscript>

The first item defined is a class which wraps around unittest tests that are found. It provides a mechanism to add listeners (callbacks) for the end of a test. This is how the GUI is updated when a test completes (color change for pass/fail status, etc.).

The second item (getTests) is a method which uses the unittest module to find all of the unittest compatible tests in the loaded modules. This is one of the key things to remember about embedding IronPython: if you can do something much easier in Python code than trying to pull out variables and call Python methods from C#, then do it. Write a nice little wrapper method that you can easily call from C# and have it do most of the work. This will save you a lot of time with trying to get things to work. Python is great at interrogating Python code, use it to its full benefit.

The last item (runTests) is the method that is called by the host application to actually run the tests. It receives, from C#, a list (Python list) of tests to run. If there is a completion callback passed in, it wraps up the test case with the TestermaticTestCase wrapper and adds it to the test suite. Then, to run the tests, its as simple as passing the list off to the TextTestRunner class from the unittest module.

So, let’s look at how all of this gets pulled together.

[<img class="size-full wp-image-119 alignleft" title="Testermatic Toolbar" src="http://earl-of-code.com/wp-content/uploads/2012/02/testermatic_toolbar.png" alt="Testermatic Toolbar" width="113" height="24" />][2]

&nbsp;

&nbsp;

The toolbar on the Testermatic application has four buttons. The first is used to load a new set of tests. It shows an OpenFileDialog and once the user selects a Python file (*.py) it will enumerate the tests in the module and populate the list of tests. It uses the method below to load the tests.

<noscript>
  <pre><code class="language-c# c#">/// &lt;summary&gt;
/// Imports the file and then searches for tests within loaded modules.
/// &lt;/summary&gt;
/// &lt;param name="file"&gt;The file to import&lt;/param&gt;
void LoadTests(string file) {
    InitializeEngine();

    ICollection&lt;string&gt; searchPaths = _engine.GetSearchPaths();
    searchPaths.Add(Path.GetDirectoryName(file));

    _engine.SetSearchPaths(searchPaths);
    string module_name = Path.GetFileNameWithoutExtension(file);

    // import the module into the ScriptScope
    Import(module_name);

    // execute the Python method in the testermatic module to get the list of tests
    List tests = _engine.Execute&lt;List&gt;("testermatic.getTests()", _scope);

    test_list.Items.Clear();
    // enumerate through the tests and add a ListViewItem for each one.
    foreach (dynamic test in tests) {
        string testName = test.id();
        Invoke((Action)delegate() {
            ListViewItem new_item = test_list.Items.Add(testName);
            new_item.Checked = true;
            new_item.Tag = test;
        });
    }

    _current_test_file = file;
    _stdout.WriteLine("Ready...");
}</code></pre>
</noscript>

The Import method is as follows

<noscript>
  <pre><code class="language-c# c#">void Import(params string[] module_names) {
    foreach (string module_name in module_names) {
         _stdout.WriteLine("importing {0}...", module_name);
         try {
             _scope.SetVariable(module_name, _engine.ImportModule(module_name));
         } catch (ImportException ex) {
             _stdout.WriteLine("Could not import {0} - {1}", module_name, ex.Message);
         } catch (Exception ex) {
             _stdout.WriteLine("Exception importing {0} - {1}", module_name, ex.Message);
         }
     }
}</code></pre>
</noscript>

The second button on the toolbar is used to actually run the tests. It creates a List (Python list) using the items in the ListView that are checked (remember the test object &#8212; the Python test object &#8212; was assigned to the .Tag property of each ListViewItem so it is easy to pull out).

<noscript>
  <pre><code class="language-c# c#">/// &lt;summary&gt;
/// Runs the tests in the given test list.
/// &lt;/summary&gt;
/// &lt;param name="test_list"&gt;&lt;/param&gt;
void RunTests(List test_list) {
    try {
        // call our utility method to run the selected tests
        ScriptScope testermatic = null;
        dynamic runTests = null;
        if (_scope.TryGetVariable("testermatic", out testermatic) &&
            boardom.TryGetVariable("runTests", out runTests)) {
            runTests(test_list, new TestsCompleteDelegate(TestsComplete));
        }
    } catch (Exception ex) {
        _stderr.WriteLine("Error running tests - {0}", ex.ToString());
    }
}</code></pre>
</noscript>

You can see that this is interacting with the Python code in a different way than was done to retrieve the list of tests. This retrieves a ScriptScope (think module) object for the testermatic Python module shown earlier. It then gets the runTests method from that module and since it is a dynamic, it can call it just like a function. The TestCompleteDelegate is the callback for when the test completes.

<noscript>
  <pre><code class="language-c# c#">delegate object TestsCompleteDelegate(object sender, int num_tests, List failures, List errors);</code></pre>
</noscript>

The TestComplete method which is called just updates the UI based on which tests passed, which tests failed and which tests had an error (green for pass, red for fail, purple for error conditions).

The third item on the toolbar is just a reload button. This is helpful if you are editing the Python script containing the tests and you want to reload to get any new tests, or new changes you made to the file. This calls the same LoadTests method that was called when the file was selected. A new ScriptEngine instance and a new ScriptScope instance are created (and cleaned up if already created) every time the LoadTests method is called.

The final button in the toolbar is a simple little email capability, because when something fails its always nice to notify people about it. The button displays an email form as shown below to allow the user to enter To and From addresses as well as a message. The output from the tests will be added after the message and sent to both the To and From addresses.

[<img class="alignnone size-full wp-image-129" title="Email Form" src="http://earl-of-code.com/wp-content/uploads/2012/02/testermatic_emailform.png" alt="Email Form" width="777" height="508" />][3]

Now, do I use the application at work to run normal Python unit tests? No, as I mentioned before, this was mainly to allow the hardware team at my work to develop simple tests to quickly triage issues with the boards. They can quickly modify the test for special circumstances and run the whole suite of tests again. It also allows them to send the report to another person so they can review the test run.

I hope this simple little application has shown you how easy it is to incorporate IronPython into your application. I attached the project for this application below so you can download it and play around with it.

[Testermatic Project][4]

<span class='st\_facebook\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='Testing With IronPython' st\_url='http://earl-of-code.com/2012/02/testing-with-ironpython/' displayText='pinterest'></span>

 [1]: http://earl-of-code.com/wp-content/uploads/2012/02/image.png
 [2]: http://earl-of-code.com/wp-content/uploads/2012/02/testermatic_toolbar.png
 [3]: http://earl-of-code.com/wp-content/uploads/2012/02/testermatic_emailform.png
 [4]: http://earl-of-code.com/wp-content/uploads/2012/02/Testermatic.zip