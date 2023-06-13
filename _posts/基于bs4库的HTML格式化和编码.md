
## 1. 基于bs4库的HTML格式输出

如何能让HTML内容更加友好的显示呢？

这里面的友好，不只是使HTML内容更容易使人阅读，也使程序能更好地读取和分析。

在bs4库中，提出了一个方法`prettify()`，我们在代码中演示。
<!--more-->
```python
>>> import requests
>>> from bs4 import BeautifulSoup
>>> url = "https://www.crummy.com/software/BeautifulSoup/"
>>> r = requests.get(url)
>>> r.status_code
200
>>> r.encoding
'UTF-8'
>>> r.text
'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN"\n"http://www.w3.org/TR/REC-html40/transitional.dtd">\n<html>\n<head>\n<meta http-equiv="Content-Type" content="text/html; charset=utf-8">\n<title>Beautiful Soup: We called him Tortoise because he taught us.</title>\n<link rev="made" href="mailto:leonardr@segfault.org">\n<link rel="stylesheet" type="text/css" href="/nb/themes/Default/nb.css">\n<meta name="Description" content="Beautiful Soup: a library designed for screen-scraping HTML and XML.">\n<meta name="generator" content="Markov Approximation 1.4 (module: leonardr)">\n<meta name="author" content="Leonard Richardson">\n</head>\n<body bgcolor="white" text="black" link="blue" vlink="660066" alink="red">\n<style>\n#tidelift { }\n\n#tidelift a {\n border: 1px solid #666666;\n margin-left: auto;\n padding: 10px;\n text-decoration: none;\n}\n\n#tidelift .cta {\n background: url("tidelift.svg") no-repeat;\n padding-left: 30px;\n}\n</style>\t\t   \n\n<img align="right" src="10.1.jpg" width="250"><br />\n\n<p>[ <a href="#Download">Download</a> | <a\nhref="bs4/doc/">Documentation</a> | <a href="#HallOfFame">Hall of Fame</a> | <a href="enterprise.html">For enterprise</a> | <a href="https://code.launchpad.net/beautifulsoup">Source</a> | <a href="https://bazaar.launchpad.net/%7Eleonardr/beautifulsoup/bs4/view/head:/CHANGELOG">Changelog</a> | <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">Discussion group</a>  | <a href="zine/">Zine</a> ]</p>\n\n<div align="center">\n\n<a href="bs4/download/"><h1>Beautiful Soup</h1></a>\n\n</div>\n\n<p>You didn\'t write that awful page. You\'re just trying to get some\ndata out of it. Beautiful Soup is here to help. Since 2004, it\'s been\nsaving programmers hours or days of work on quick-turnaround\nscreen scraping projects.</p>\n\n<p>Beautiful Soup is a Python library designed for quick turnaround\nprojects like screen-scraping. Three features make it powerful:\n\n<ol>\n\n<li>Beautiful Soup provides a few simple methods and Pythonic idioms\nfor navigating, searching, and modifying a parse tree: a toolkit for\ndissecting a document and extracting what you need. It doesn\'t take\nmuch code to write an application\n\n<li>Beautiful Soup automatically converts incoming documents to\nUnicode and outgoing documents to UTF-8. You don\'t have to think\nabout encodings, unless the document doesn\'t specify an encoding and\nBeautiful Soup can\'t detect one. Then you just have to specify the\noriginal encoding.\n\n<li>Beautiful Soup sits on top of popular Python parsers like <a\nhref="http://lxml.de/">lxml</a> and <a\nhref="http://code.google.com/p/html5lib/">html5lib</a>, allowing you\nto try out different parsing strategies or trade speed for\nflexibility.\n\n</ol>\n\n<p>Beautiful Soup parses anything you give it, and does the tree\ntraversal stuff for you. You can tell it "Find all the links", or\n"Find all the links of class <tt>externalLink</tt>", or "Find all the\nlinks whose urls match "foo.com", or "Find the table heading that\'s\ngot bold text, then give me that text."\n\n<p>Valuable data that was once locked up in poorly-designed websites\nis now within your reach. Projects that would have taken hours take\nonly minutes with Beautiful Soup.\n\n<p>Interested? <a href="bs4/doc/">Read more.</a>\n\n<h3>Getting and giving support</h3>\n\n<div id="tidelift" align="center">\n<a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&utm_medium=referral&utm_campaign=enterprise" target="_blank">\n <span class="cta">\n  Beautiful Soup for enterprise available via Tidelift\n </span>\n</a>\n</div>\n\n<p>If you have questions, send them to <a\nhref="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">the discussion\ngroup</a>. If you find a bug, <a href="https://bugs.launchpad.net/beautifulsoup/">file it on Launchpad</a>. If it\'s a security vulnerability, report it confidentially through <a href="https://tidelift.com/security">Tidelift</a>.</p>\n\n<p>If you use Beautiful Soup as part of your work, please consider a <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&utm_medium=referral&utm_campaign=website">Tidelift subscription</a>. This will support many of the free software projects your organization depends on, not just Beautiful Soup.\n\n\n<p>If Beautiful Soup is useful to you on a personal level, you might like to read <a href="zine/"><i>Tool Safety</i></a>, a short zine I wrote about what I learned about software development from working on Beautiful Soup. Thanks!</p>\n</div>\n\n\n<a name="Download"><h2>Download Beautiful Soup</h2></a>\n\n<p>The current release is <a href="bs4/download/">Beautiful Soup\n4.8.2</a> (December 24, 2019). You can install Beautiful Soup 4 with\n<code>pip install beautifulsoup4</code>.\n\n<p>In Debian and Ubuntu, Beautiful Soup is available as the\n<code>python-bs4</code> package (for Python 2) or the\n<code>python3-bs4</code> package (for Python 3). In Fedora it\'s\navailable as the <code>python-beautifulsoup4</code> package.\n\n<p>Beautiful Soup is licensed under the MIT license, so you can also\ndownload the tarball, drop the <code>bs4/</code> directory into almost\nany Python application (or into your library path) and start using it\nimmediately. (If you want to do this under Python 3, you will need to\nmanually convert the code using <code>2to3</code>.)\n\n<p>Beautiful Soup 4 works on both Python 2 (2.7+) and Python\n3. Support for Python 2 will be discontinued on or after December 31,\n2020&mdash;one year after the Python 2 sunsetting date.\n\n<h3>Beautiful Soup 3</h3>\n\n<p>Beautiful Soup 3 was the official release line of Beautiful Soup\nfrom May 2006 to March 2012. It does not support Python 3 and it will\nbe discontinued on or after December 31, 2020&mdash;one year after the\nPython 2 sunsetting date. If you have any active projects using\nBeautiful Soup 3, you should migrate to Beautiful Soup 4 as part of\nyour Python 3 conversion.\n\n<p><a\nhref="http://www.crummy.com/software/BeautifulSoup/bs3/documentation.html">Here\'s\nthe Beautiful Soup 3 documentation.</a>\n\n<p>The current and hopefully final release of Beautiful Soup 3 is <a\nhref="download/3.x/BeautifulSoup-3.2.2.tar.gz">3.2.2</a> (October 5,\n2019). It\'s the <code>BeautifulSoup</code> package on pip. It\'s also\navailable as <code>python-beautifulsoup</code> in Debian and Ubuntu,\nand as <code>python-BeautifulSoup</code> in Fedora.\n\n<p>Once Beautiful Soup 3 is discontinued, these package names will be available for use by a more recent version of Beautiful Soup.\n\n<p>Beautiful Soup 3, like Beautiful Soup 4, is <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup?utm_source=pypi-beautifulsoup&utm_medium=referral&utm_campaign=website">supported through Tidelift</a>.</p>\n\n<a name="HallOfFame"><h2>Hall of Fame</h2></a>\n\n<p>Over the years, Beautiful Soup has been used in hundreds of\ndifferent projects. There\'s no way I can list them all, but I want to\nhighlight a few high-profile projects. Beautiful Soup isn\'t what makes\nthese projects interesting, but it did make their completion easier:\n\n<ul>\n\n<li><a\n href="http://www.nytimes.com/2007/10/25/arts/design/25vide.html">"Movable\n Type"</a>, a work of digital art on display in the lobby of the New\n York Times building, uses Beautiful Soup to scrape news feeds.\n\n<li>Reddit uses Beautiful Soup to <a\nhref="https://github.com/reddit/reddit/blob/85f9cff3e2ab9bb8f19b96acd8da4ebacc079f04/r2/r2/lib/media.py">parse\na page that\'s been linked to and find a representative image</a>.\n\n<li>Alexander Harrowell uses Beautiful Soup to <a\n href="http://www.harrowell.org.uk/viktormap.html">track the business\n activities</a> of an arms merchant.\n\n<li>The developers of Python itself used Beautiful Soup to <a\nhref="http://svn.python.org/view/tracker/importer/">migrate the Python\nbug tracker from Sourceforge to Roundup</a>.\n\n<li>The <a href="http://www2.ljworld.com/">Lawrence Journal-World</a>\nuses Beautiful Soup to <A\nhref="http://www.b-list.org/weblog/2010/nov/02/news-done-broke/">gather\nstatewide election results</a>.\n\n<li>The <a href="http://esrl.noaa.gov/gsd/fab/">NOAA\'s Forecast\nApplications Branch</a> uses Beautiful Soup in <a\nhref="http://laps.noaa.gov/topograbber/">TopoGrabber</a>, a script for\ndownloading "high resolution USGS datasets."\n\n</ul>\n\n<p>If you\'ve used Beautiful Soup in a project you\'d like me to know\nabout, please do send email to me or <a\nhref="http://groups.google.com/group/beautifulsoup/">the discussion\ngroup</a>.\n\n<h2>Development</h2>\n\n<p>Development happens at <a\nhref="https://launchpad.net/beautifulsoup">Launchpad</a>. You can <a\nhref="https://code.launchpad.net/beautifulsoup/">get the source\ncode</a> or <a href="https://bugs.launchpad.net/beautifulsoup/">file\nbugs</a>.<hr><table><tr><td valign="top">\n<p>This document (<a href="/source/software/BeautifulSoup/index.bhtml">source</a>) is part of Crummy, the webspace of <a href="/self/">Leonard Richardson</a> (<a href="/self/contact.html">contact information</a>). It was last modified on Friday, January 31 2020, 13:44:05 Nowhere Standard Time and last built on Saturday, March 07 2020, 09:00:01 Nowhere Standard Time.</p><p><table class="licenseText"><tr><td><a href="http://creativecommons.org/licenses/by-sa/2.0/"><img border="0" src="/nb//resources/img/somerights20.jpg"></a></td><td valign="top">Crummy is &copy; 1996-2020 Leonard Richardson. Unless otherwise noted, all text licensed under a <a href="http://creativecommons.org/licenses/by-sa/2.0/">Creative Commons License</a>.</td></tr></table></span><!--<rdf:RDF xmlns="http://web.resource.org/cc/" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"><Work rdf:about="http://www.crummy.com/"><dc:title>Crummy: The Site</dc:title><dc:rights><Agent><dc:title>Crummy: the Site</dc:title></Agent></dc:rights><dc:format>text/html</dc:format><license rdf:resource=http://creativecommons.org/licenses/by-sa/2.0//></Work><License rdf:about="http://creativecommons.org/licenses/by-sa/2.0/"></License></rdf:RDF>--></p></td><td valign=top><p><b>Document tree:</b>\n<dl><dd><a href="http://www.crummy.com/">http://www.crummy.com/</a><dl><dd><a href="http://www.crummy.com/software/">software/</a><dl><dd><a href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup/</a></dl>\n</dl>\n</dl>\n\n\nSite Search:\n\n<form method="get" action="/search/">\n        <input type="text" name="q" maxlength="255" value=""></input>\n        </form>\n        </td>\n\n</tr>\n\n</table>\n</body>\n</html>\n'
```

看上面代码我们可以发现，我们得到的HTML文本是没有格式可言的，所以信息全在一行显示。

使用bs4库中的方法`prettify()`后代码如下。

```python
>>> soup = BeautifulSoup(r.text, "html.parser")
>>> soup.prettify()
'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN"\n"http://www.w3.org/TR/REC-html40/transitional.dtd">\n<html>\n <head>\n  <meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>\n  <title>\n   Beautiful Soup: We called him Tortoise because he taught us.\n  </title>\n  <link href="mailto:leonardr@segfault.org" rev="made"/>\n  <link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>\n  <meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>\n  <meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>\n  <meta content="Leonard Richardson" name="author"/>\n </head>\n <body alink="red" bgcolor="white" link="blue" text="black" vlink="660066">\n  <style>\n   #tidelift { }\n\n#tidelift a {\n border: 1px solid #666666;\n margin-left: auto;\n padding: 10px;\n text-decoration: none;\n}\n\n#tidelift .cta {\n background: url("tidelift.svg") no-repeat;\n padding-left: 30px;\n}\n  </style>\n  <img align="right" src="10.1.jpg" width="250"/>\n  <br/>\n  <p>\n   [\n   <a href="#Download">\n    Download\n   </a>\n   |\n   <a href="bs4/doc/">\n    Documentation\n   </a>\n   |\n   <a href="#HallOfFame">\n    Hall of Fame\n   </a>\n   |\n   <a href="enterprise.html">\n    For enterprise\n   </a>\n   |\n   <a href="https://code.launchpad.net/beautifulsoup">\n    Source\n   </a>\n   |\n   <a href="https://bazaar.launchpad.net/%7Eleonardr/beautifulsoup/bs4/view/head:/CHANGELOG">\n    Changelog\n   </a>\n   |\n   <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">\n    Discussion group\n   </a>\n   |\n   <a href="zine/">\n    Zine\n   </a>\n   ]\n  </p>\n  <div align="center">\n   <a href="bs4/download/">\n    <h1>\n     Beautiful Soup\n    </h1>\n   </a>\n  </div>\n  <p>\n   You didn\'t write that awful page. You\'re just trying to get some\ndata out of it. Beautiful Soup is here to help. Since 2004, it\'s been\nsaving programmers hours or days of work on quick-turnaround\nscreen scraping projects.\n  </p>\n  <p>\n   Beautiful Soup is a Python library designed for quick turnaround\nprojects like screen-scraping. Three features make it powerful:\n   <ol>\n    <li>\n     Beautiful Soup provides a few simple methods and Pythonic idioms\nfor navigating, searching, and modifying a parse tree: a toolkit for\ndissecting a document and extracting what you need. It doesn\'t take\nmuch code to write an application\n     <li>\n      Beautiful Soup automatically converts incoming documents to\nUnicode and outgoing documents to UTF-8. You don\'t have to think\nabout encodings, unless the document doesn\'t specify an encoding and\nBeautiful Soup can\'t detect one. Then you just have to specify the\noriginal encoding.\n      <li>\n       Beautiful Soup sits on top of popular Python parsers like\n       <a href="http://lxml.de/">\n        lxml\n       </a>\n       and\n       <a href="http://code.google.com/p/html5lib/">\n        html5lib\n       </a>\n       , allowing you\nto try out different parsing strategies or trade speed for\nflexibility.\n      </li>\n     </li>\n    </li>\n   </ol>\n   <p>\n    Beautiful Soup parses anything you give it, and does the tree\ntraversal stuff for you. You can tell it "Find all the links", or\n"Find all the links of class\n    <tt>\n     externalLink\n    </tt>\n    ", or "Find all the\nlinks whose urls match "foo.com", or "Find the table heading that\'s\ngot bold text, then give me that text."\n    <p>\n     Valuable data that was once locked up in poorly-designed websites\nis now within your reach. Projects that would have taken hours take\nonly minutes with Beautiful Soup.\n     <p>\n      Interested?\n      <a href="bs4/doc/">\n       Read more.\n      </a>\n      <h3>\n       Getting and giving support\n      </h3>\n      <div align="center" id="tidelift">\n       <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&amp;utm_medium=referral&amp;utm_campaign=enterprise" target="_blank">\n        <span class="cta">\n         Beautiful Soup for enterprise available via Tidelift\n        </span>\n       </a>\n      </div>\n      <p>\n       If you have questions, send them to\n       <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">\n        the discussion\ngroup\n       </a>\n       . If you find a bug,\n       <a href="https://bugs.launchpad.net/beautifulsoup/">\n        file it on Launchpad\n       </a>\n       . If it\'s a security vulnerability, report it confidentially through\n       <a href="https://tidelift.com/security">\n        Tidelift\n       </a>\n       .\n      </p>\n      <p>\n       If you use Beautiful Soup as part of your work, please consider a\n       <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&amp;utm_medium=referral&amp;utm_campaign=website">\n        Tidelift subscription\n       </a>\n       . This will support many of the free software projects your organization depends on, not just Beautiful Soup.\n       <p>\n        If Beautiful Soup is useful to you on a personal level, you might like to read\n        <a href="zine/">\n         <i>\n          Tool Safety\n         </i>\n        </a>\n        , a short zine I wrote about what I learned about software development from working on Beautiful Soup. Thanks!\n       </p>\n      </p>\n     </p>\n    </p>\n   </p>\n  </p>\n </body>\n</html>\n<a name="Download">\n <h2>\n  Download Beautiful Soup\n </h2>\n</a>\n<p>\n The current release is\n <a href="bs4/download/">\n  Beautiful Soup\n4.8.2\n </a>\n (December 24, 2019). You can install Beautiful Soup 4 with\n <code>\n  pip install beautifulsoup4\n </code>\n .\n <p>\n  In Debian and Ubuntu, Beautiful Soup is available as the\n  <code>\n   python-bs4\n  </code>\n  package (for Python 2) or the\n  <code>\n   python3-bs4\n  </code>\n  package (for Python 3). In Fedora it\'s\navailable as the\n  <code>\n   python-beautifulsoup4\n  </code>\n  package.\n  <p>\n   Beautiful Soup is licensed under the MIT license, so you can also\ndownload the tarball, drop the\n   <code>\n    bs4/\n   </code>\n   directory into almost\nany Python application (or into your library path) and start using it\nimmediately. (If you want to do this under Python 3, you will need to\nmanually convert the code using\n   <code>\n    2to3\n   </code>\n   .)\n   <p>\n    Beautiful Soup 4 works on both Python 2 (2.7+) and Python\n3. Support for Python 2 will be discontinued on or after December 31,\n2020—one year after the Python 2 sunsetting date.\n    <h3>\n     Beautiful Soup 3\n    </h3>\n    <p>\n     Beautiful Soup 3 was the official release line of Beautiful Soup\nfrom May 2006 to March 2012. It does not support Python 3 and it will\nbe discontinued on or after December 31, 2020—one year after the\nPython 2 sunsetting date. If you have any active projects using\nBeautiful Soup 3, you should migrate to Beautiful Soup 4 as part of\nyour Python 3 conversion.\n     <p>\n      <a href="http://www.crummy.com/software/BeautifulSoup/bs3/documentation.html">\n       Here\'s\nthe Beautiful Soup 3 documentation.\n      </a>\n      <p>\n       The current and hopefully final release of Beautiful Soup 3 is\n       <a href="download/3.x/BeautifulSoup-3.2.2.tar.gz">\n        3.2.2\n       </a>\n       (October 5,\n2019). It\'s the\n       <code>\n        BeautifulSoup\n       </code>\n       package on pip. It\'s also\navailable as\n       <code>\n        python-beautifulsoup\n       </code>\n       in Debian and Ubuntu,\nand as\n       <code>\n        python-BeautifulSoup\n       </code>\n       in Fedora.\n       <p>\n        Once Beautiful Soup 3 is discontinued, these package names will be available for use by a more recent version of Beautiful Soup.\n        <p>\n         Beautiful Soup 3, like Beautiful Soup 4, is\n         <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup?utm_source=pypi-beautifulsoup&amp;utm_medium=referral&amp;utm_campaign=website">\n          supported through Tidelift\n         </a>\n         .\n        </p>\n        <a name="HallOfFame">\n         <h2>\n          Hall of Fame\n         </h2>\n        </a>\n        <p>\n         Over the years, Beautiful Soup has been used in hundreds of\ndifferent projects. There\'s no way I can list them all, but I want to\nhighlight a few high-profile projects. Beautiful Soup isn\'t what makes\nthese projects interesting, but it did make their completion easier:\n         <ul>\n          <li>\n           <a href="http://www.nytimes.com/2007/10/25/arts/design/25vide.html">\n            "Movable\n Type"\n           </a>\n           , a work of digital art on display in the lobby of the New\n York Times building, uses Beautiful Soup to scrape news feeds.\n           <li>\n            Reddit uses Beautiful Soup to\n            <a href="https://github.com/reddit/reddit/blob/85f9cff3e2ab9bb8f19b96acd8da4ebacc079f04/r2/r2/lib/media.py">\n             parse\na page that\'s been linked to and find a representative image\n            </a>\n            .\n            <li>\n             Alexander Harrowell uses Beautiful Soup to\n             <a href="http://www.harrowell.org.uk/viktormap.html">\n              track the business\n activities\n             </a>\n             of an arms merchant.\n             <li>\n              The developers of Python itself used Beautiful Soup to\n              <a href="http://svn.python.org/view/tracker/importer/">\n               migrate the Python\nbug tracker from Sourceforge to Roundup\n              </a>\n              .\n              <li>\n               The\n               <a href="http://www2.ljworld.com/">\n                Lawrence Journal-World\n               </a>\n               uses Beautiful Soup to\n               <a href="http://www.b-list.org/weblog/2010/nov/02/news-done-broke/">\n                gather\nstatewide election results\n               </a>\n               .\n               <li>\n                The\n                <a href="http://esrl.noaa.gov/gsd/fab/">\n                 NOAA\'s Forecast\nApplications Branch\n                </a>\n                uses Beautiful Soup in\n                <a href="http://laps.noaa.gov/topograbber/">\n                 TopoGrabber\n                </a>\n                , a script for\ndownloading "high resolution USGS datasets."\n               </li>\n              </li>\n             </li>\n            </li>\n           </li>\n          </li>\n         </ul>\n         <p>\n          If you\'ve used Beautiful Soup in a project you\'d like me to know\nabout, please do send email to me or\n          <a href="http://groups.google.com/group/beautifulsoup/">\n           the discussion\ngroup\n          </a>\n          .\n          <h2>\n           Development\n          </h2>\n          <p>\n           Development happens at\n           <a href="https://launchpad.net/beautifulsoup">\n            Launchpad\n           </a>\n           . You can\n           <a href="https://code.launchpad.net/beautifulsoup/">\n            get the source\ncode\n           </a>\n           or\n           <a href="https://bugs.launchpad.net/beautifulsoup/">\n            file\nbugs\n           </a>\n           .\n           <hr/>\n           <table>\n            <tr>\n             <td valign="top">\n              <p>\n               This document (\n               <a href="/source/software/BeautifulSoup/index.bhtml">\n                source\n               </a>\n               ) is part of Crummy, the webspace of\n               <a href="/self/">\n                Leonard Richardson\n               </a>\n               (\n               <a href="/self/contact.html">\n                contact information\n               </a>\n               ). It was last modified on Friday, January 31 2020, 13:44:05 Nowhere Standard Time and last built on Saturday, March 07 2020, 09:00:01 Nowhere Standard Time.\n              </p>\n              <p>\n               <table class="licenseText">\n                <tr>\n                 <td>\n                  <a href="http://creativecommons.org/licenses/by-sa/2.0/">\n                   <img border="0" src="/nb//resources/img/somerights20.jpg"/>\n                  </a>\n                 </td>\n                 <td valign="top">\n                  Crummy is © 1996-2020 Leonard Richardson. Unless otherwise noted, all text licensed under a\n                  <a href="http://creativecommons.org/licenses/by-sa/2.0/">\n                   Creative Commons License\n                  </a>\n                  .\n                 </td>\n                </tr>\n               </table>\n              </p>\n             </td>\n            </tr>\n           </table>\n          </p>\n         </p>\n        </p>\n       </p>\n      </p>\n     </p>\n    </p>\n   </p>\n  </p>\n </p>\n</p>\n<!--<rdf:RDF xmlns="http://web.resource.org/cc/" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"><Work rdf:about="http://www.crummy.com/"><dc:title>Crummy: The Site</dc:title><dc:rights><Agent><dc:title>Crummy: the Site</dc:title></Agent></dc:rights><dc:format>text/html</dc:format><license rdf:resource=http://creativecommons.org/licenses/by-sa/2.0//></Work><License rdf:about="http://creativecommons.org/licenses/by-sa/2.0/"></License></rdf:RDF>-->\n<td valign="top">\n <p>\n  <b>\n   Document tree:\n  </b>\n  <dl>\n   <dd>\n    <a href="http://www.crummy.com/">\n     http://www.crummy.com/\n    </a>\n    <dl>\n     <dd>\n      <a href="http://www.crummy.com/software/">\n       software/\n      </a>\n      <dl>\n       <dd>\n        <a href="http://www.crummy.com/software/BeautifulSoup/">\n         BeautifulSoup/\n        </a>\n       </dd>\n      </dl>\n     </dd>\n    </dl>\n   </dd>\n  </dl>\n  Site Search:\n  <form action="/search/" method="get">\n   <input maxlength="255" name="q" type="text" value=""/>\n  </form>\n </p>\n</td>\n'
```

观察代码可以看到，方法`prettify()`在每个标签后和每个标签内融后，都加了一个换行符`\n'`。

<!--more-->

下面我们用`print`打印出这个HTML文本，看看效果。

```HTML
>>> print(soup.prettify())
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN"
"http://www.w3.org/TR/REC-html40/transitional.dtd">
<html>
 <head>
  <meta content="text/html; charset=utf-8" http-equiv="Content-Type"/>
  <title>
   Beautiful Soup: We called him Tortoise because he taught us.
  </title>
  <link href="mailto:leonardr@segfault.org" rev="made"/>
  <link href="/nb/themes/Default/nb.css" rel="stylesheet" type="text/css"/>
  <meta content="Beautiful Soup: a library designed for screen-scraping HTML and XML." name="Description"/>
  <meta content="Markov Approximation 1.4 (module: leonardr)" name="generator"/>
  <meta content="Leonard Richardson" name="author"/>
 </head>
 <body alink="red" bgcolor="white" link="blue" text="black" vlink="660066">
  <style>
   #tidelift { }

#tidelift a {
 border: 1px solid #666666;
 margin-left: auto;
 padding: 10px;
 text-decoration: none;
}

#tidelift .cta {
 background: url("tidelift.svg") no-repeat;
 padding-left: 30px;
}
  </style>
  <img align="right" src="10.1.jpg" width="250"/>
  <br/>
  <p>
   [
   <a href="#Download">
    Download
   </a>
   |
   <a href="bs4/doc/">
    Documentation
   </a>
   |
   <a href="#HallOfFame">
    Hall of Fame
   </a>
   |
   <a href="enterprise.html">
    For enterprise
   </a>
   |
   <a href="https://code.launchpad.net/beautifulsoup">
    Source
   </a>
   |
   <a href="https://bazaar.launchpad.net/%7Eleonardr/beautifulsoup/bs4/view/head:/CHANGELOG">
    Changelog
   </a>
   |
   <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">
    Discussion group
   </a>
   |
   <a href="zine/">
    Zine
   </a>
   ]
  </p>
  <div align="center">
   <a href="bs4/download/">
    <h1>
     Beautiful Soup
    </h1>
   </a>
  </div>
  <p>
   You didn't write that awful page. You're just trying to get some
data out of it. Beautiful Soup is here to help. Since 2004, it's been
saving programmers hours or days of work on quick-turnaround
screen scraping projects.
  </p>
  <p>
   Beautiful Soup is a Python library designed for quick turnaround
projects like screen-scraping. Three features make it powerful:
   <ol>
    <li>
     Beautiful Soup provides a few simple methods and Pythonic idioms
for navigating, searching, and modifying a parse tree: a toolkit for
dissecting a document and extracting what you need. It doesn't take
much code to write an application
     <li>
      Beautiful Soup automatically converts incoming documents to
Unicode and outgoing documents to UTF-8. You don't have to think
about encodings, unless the document doesn't specify an encoding and
Beautiful Soup can't detect one. Then you just have to specify the
original encoding.
      <li>
       Beautiful Soup sits on top of popular Python parsers like
       <a href="http://lxml.de/">
        lxml
       </a>
       and
       <a href="http://code.google.com/p/html5lib/">
        html5lib
       </a>
       , allowing you
to try out different parsing strategies or trade speed for
flexibility.
      </li>
     </li>
    </li>
   </ol>
   <p>
    Beautiful Soup parses anything you give it, and does the tree
traversal stuff for you. You can tell it "Find all the links", or
"Find all the links of class
    <tt>
     externalLink
    </tt>
    ", or "Find all the
links whose urls match "foo.com", or "Find the table heading that's
got bold text, then give me that text."
    <p>
     Valuable data that was once locked up in poorly-designed websites
is now within your reach. Projects that would have taken hours take
only minutes with Beautiful Soup.
     <p>
      Interested?
      <a href="bs4/doc/">
       Read more.
      </a>
      <h3>
       Getting and giving support
      </h3>
      <div align="center" id="tidelift">
       <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&amp;utm_medium=referral&amp;utm_campaign=enterprise" target="_blank">
        <span class="cta">
         Beautiful Soup for enterprise available via Tidelift
        </span>
       </a>
      </div>
      <p>
       If you have questions, send them to
       <a href="https://groups.google.com/forum/?fromgroups#!forum/beautifulsoup">
        the discussion
group
       </a>
       . If you find a bug,
       <a href="https://bugs.launchpad.net/beautifulsoup/">
        file it on Launchpad
       </a>
       . If it's a security vulnerability, report it confidentially through
       <a href="https://tidelift.com/security">
        Tidelift
       </a>
       .
      </p>
      <p>
       If you use Beautiful Soup as part of your work, please consider a
       <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup4?utm_source=pypi-beautifulsoup4&amp;utm_medium=referral&amp;utm_campaign=website">
        Tidelift subscription
       </a>
       . This will support many of the free software projects your organization depends on, not just Beautiful Soup.
       <p>
        If Beautiful Soup is useful to you on a personal level, you might like to read
        <a href="zine/">
         <i>
          Tool Safety
         </i>
        </a>
        , a short zine I wrote about what I learned about software development from working on Beautiful Soup. Thanks!
       </p>
      </p>
     </p>
    </p>
   </p>
  </p>
 </body>
</html>
<a name="Download">
 <h2>
  Download Beautiful Soup
 </h2>
</a>
<p>
 The current release is
 <a href="bs4/download/">
  Beautiful Soup
4.8.2
 </a>
 (December 24, 2019). You can install Beautiful Soup 4 with
 <code>
  pip install beautifulsoup4
 </code>
 .
 <p>
  In Debian and Ubuntu, Beautiful Soup is available as the
  <code>
   python-bs4
  </code>
  package (for Python 2) or the
  <code>
   python3-bs4
  </code>
  package (for Python 3). In Fedora it's
available as the
  <code>
   python-beautifulsoup4
  </code>
  package.
  <p>
   Beautiful Soup is licensed under the MIT license, so you can also
download the tarball, drop the
   <code>
    bs4/
   </code>
   directory into almost
any Python application (or into your library path) and start using it
immediately. (If you want to do this under Python 3, you will need to
manually convert the code using
   <code>
    2to3
   </code>
   .)
   <p>
    Beautiful Soup 4 works on both Python 2 (2.7+) and Python
3. Support for Python 2 will be discontinued on or after December 31,
2020—one year after the Python 2 sunsetting date.
    <h3>
     Beautiful Soup 3
    </h3>
    <p>
     Beautiful Soup 3 was the official release line of Beautiful Soup
from May 2006 to March 2012. It does not support Python 3 and it will
be discontinued on or after December 31, 2020—one year after the
Python 2 sunsetting date. If you have any active projects using
Beautiful Soup 3, you should migrate to Beautiful Soup 4 as part of
your Python 3 conversion.
     <p>
      <a href="http://www.crummy.com/software/BeautifulSoup/bs3/documentation.html">
       Here's
the Beautiful Soup 3 documentation.
      </a>
      <p>
       The current and hopefully final release of Beautiful Soup 3 is
       <a href="download/3.x/BeautifulSoup-3.2.2.tar.gz">
        3.2.2
       </a>
       (October 5,
2019). It's the
       <code>
        BeautifulSoup
       </code>
       package on pip. It's also
available as
       <code>
        python-beautifulsoup
       </code>
       in Debian and Ubuntu,
and as
       <code>
        python-BeautifulSoup
       </code>
       in Fedora.
       <p>
        Once Beautiful Soup 3 is discontinued, these package names will be available for use by a more recent version of Beautiful Soup.
        <p>
         Beautiful Soup 3, like Beautiful Soup 4, is
         <a href="https://tidelift.com/subscription/pkg/pypi-beautifulsoup?utm_source=pypi-beautifulsoup&amp;utm_medium=referral&amp;utm_campaign=website">
          supported through Tidelift
         </a>
         .
        </p>
        <a name="HallOfFame">
         <h2>
          Hall of Fame
         </h2>
        </a>
        <p>
         Over the years, Beautiful Soup has been used in hundreds of
different projects. There's no way I can list them all, but I want to
highlight a few high-profile projects. Beautiful Soup isn't what makes
these projects interesting, but it did make their completion easier:
         <ul>
          <li>
           <a href="http://www.nytimes.com/2007/10/25/arts/design/25vide.html">
            "Movable
 Type"
           </a>
           , a work of digital art on display in the lobby of the New
 York Times building, uses Beautiful Soup to scrape news feeds.
           <li>
            Reddit uses Beautiful Soup to
            <a href="https://github.com/reddit/reddit/blob/85f9cff3e2ab9bb8f19b96acd8da4ebacc079f04/r2/r2/lib/media.py">
             parse
a page that's been linked to and find a representative image
            </a>
            .
            <li>
             Alexander Harrowell uses Beautiful Soup to
             <a href="http://www.harrowell.org.uk/viktormap.html">
              track the business
 activities
             </a>
             of an arms merchant.
             <li>
              The developers of Python itself used Beautiful Soup to
              <a href="http://svn.python.org/view/tracker/importer/">
               migrate the Python
bug tracker from Sourceforge to Roundup
              </a>
              .
              <li>
               The
               <a href="http://www2.ljworld.com/">
                Lawrence Journal-World
               </a>
               uses Beautiful Soup to
               <a href="http://www.b-list.org/weblog/2010/nov/02/news-done-broke/">
                gather
statewide election results
               </a>
               .
               <li>
                The
                <a href="http://esrl.noaa.gov/gsd/fab/">
                 NOAA's Forecast
Applications Branch
                </a>
                uses Beautiful Soup in
                <a href="http://laps.noaa.gov/topograbber/">
                 TopoGrabber
                </a>
                , a script for
downloading "high resolution USGS datasets."
               </li>
              </li>
             </li>
            </li>
           </li>
          </li>
         </ul>
         <p>
          If you've used Beautiful Soup in a project you'd like me to know
about, please do send email to me or
          <a href="http://groups.google.com/group/beautifulsoup/">
           the discussion
group
          </a>
          .
          <h2>
           Development
          </h2>
          <p>
           Development happens at
           <a href="https://launchpad.net/beautifulsoup">
            Launchpad
           </a>
           . You can
           <a href="https://code.launchpad.net/beautifulsoup/">
            get the source
code
           </a>
           or
           <a href="https://bugs.launchpad.net/beautifulsoup/">
            file
bugs
           </a>
           .
           <hr/>
           <table>
            <tr>
             <td valign="top">
              <p>
               This document (
               <a href="/source/software/BeautifulSoup/index.bhtml">
                source
               </a>
               ) is part of Crummy, the webspace of
               <a href="/self/">
                Leonard Richardson
               </a>
               (
               <a href="/self/contact.html">
                contact information
               </a>
               ). It was last modified on Friday, January 31 2020, 13:44:05 Nowhere Standard Time and last built on Saturday, March 07 2020, 09:00:01 Nowhere Standard Time.
              </p>
              <p>
               <table class="licenseText">
                <tr>
                 <td>
                  <a href="http://creativecommons.org/licenses/by-sa/2.0/">
                   <img border="0" src="/nb//resources/img/somerights20.jpg"/>
                  </a>
                 </td>
                 <td valign="top">
                  Crummy is © 1996-2020 Leonard Richardson. Unless otherwise noted, all text licensed under a
                  <a href="http://creativecommons.org/licenses/by-sa/2.0/">
                   Creative Commons License
                  </a>
                  .
                 </td>
                </tr>
               </table>
              </p>
             </td>
            </tr>
           </table>
          </p>
         </p>
        </p>
       </p>
      </p>
     </p>
    </p>
   </p>
  </p>
 </p>
</p>
<!--<rdf:RDF xmlns="http://web.resource.org/cc/" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"><Work rdf:about="http://www.crummy.com/"><dc:title>Crummy: The Site</dc:title><dc:rights><Agent><dc:title>Crummy: the Site</dc:title></Agent></dc:rights><dc:format>text/html</dc:format><license rdf:resource=http://creativecommons.org/licenses/by-sa/2.0//></Work><License rdf:about="http://creativecommons.org/licenses/by-sa/2.0/"></License></rdf:RDF>-->
<td valign="top">
 <p>
  <b>
   Document tree:
  </b>
  <dl>
   <dd>
    <a href="http://www.crummy.com/">
     http://www.crummy.com/
    </a>
    <dl>
     <dd>
      <a href="http://www.crummy.com/software/">
       software/
      </a>
      <dl>
       <dd>
        <a href="http://www.crummy.com/software/BeautifulSoup/">
         BeautifulSoup/
        </a>
       </dd>
      </dl>
     </dd>
    </dl>
   </dd>
  </dl>
  Site Search:
  <form action="/search/" method="get">
   <input maxlength="255" name="q" type="text" value=""/>
  </form>
 </p>
</td>

```

现在的HTML文本无论是对人，还是对程序，都已经变得十分又好了。

`prettify()`方法不仅对整个HTML文本有效，对其中的标签也是有效的，观察下面代码即可。

```python
>>> print(soup.title.prettify())
<title>
 Beautiful Soup: We called him Tortoise because he taught us.
</title>
```

`<title>...</title>`标签和其内容被清晰漂亮的打印了出来。

在我们实际调试bs4库中，`prettify()`方法能起到很好的辅助作用，我们应该掌握。



## 2. 基于bs4库的HTML编码

bs4库中，将任何读入的HTML文件或字符串都转换成了`utf-8`编码。

我们知道`utf-8`编码是一种国际通用的标准编码格式，能很好的支持中文等第三国语言。

由于Python3.x系列默认支持编码是`utf-8`，因此在做相关解析的时候，bs4没有障碍。

```python
>>> soup = BeautifulSoup('<p>中文文本</p>', 'html.parser')
>>> soup.p.string
'中文文本'
>>> print(soup.p.prettify())
<p>
 中文文本
</p>
```

