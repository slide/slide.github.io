---
title: IronPython Bytecode Interpreter
author: alex
layout: post
permalink: /2013/05/ironpython-bytecode-interpreter/
categories:
  - Uncategorized
---
One of the things that people would really like to have on IronPython is support for pre-compiled Python (.pyc) files. These files are pre-parsed and converted into the bytecode format that the Python interpreter actually runs. This speeds execution up for a couple of reasons:

  1. The application does not need to parse the source code prior to execution
  2. The operations are usually at least minimally optimized already by the parsing engine

I was intrigued by the Python bytecode, it looked like a fun little thing to play around with, so I developed very minimal and early support for bytecode interpretation in the IronPython codebase. You can see my commits at my [fork of the IronPython repository on Github][1]. There is still a LOT of work that needs to be done, but there are a lot smarter people out there who once the basic framework is in place can take it and run with it.

The commits above also add support for the ever important \_\_hello\_\_ and \_\_phello\_\_ imports. One more thing making IronPython more compatible with CPython.

<span class='st\_facebook\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='facebook'></span><span class='st\_twitter\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='twitter'></span><span class='st\_linkedin\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='linkedin'></span><span class='st\_email\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='email'></span><span class='st\_sharethis\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='sharethis'></span><span class='st\_fblike\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='fblike'></span><span class='st\_plusone\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='plusone'></span><span class='st\_pinterest\_vcount' st\_title='IronPython Bytecode Interpreter' st\_url='http://earl-of-code.com/2013/05/ironpython-bytecode-interpreter/' displayText='pinterest'></span>

 [1]: https://github.com/slide/IronLanguages/commits/bytecode "bytecode branch"