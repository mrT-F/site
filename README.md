a site
=======

This is essentially forked (fully re-licensed under the wtfpl of course) from: <https://nicolas.perriault.net/code/2012/dead-easy-yet-powerful-static-website-generator-with-flask/>, as I liked the design and concept and wanted a quick way to get a website up. (It's somewhat diverged since then).

Installation/Requirements
------------
Maybe I'll manage this better one day, but get python >3.6 and:

```
    $ git clone https://github.com/mrt-f/site.git
    $ cd site
    $ python3 -m venv .site-venv/
    $ . .site-venv/bin/activate
    (.site-venv)$ pip install -r requirements.txt
```

Usage
-----

The `site-tool` command is what you want:

To serve the website locally (optionaly in `DEBUG=True` mode):

    $ ./site-tool serve --debug
    * Running on http://127.0.0.1:5000/
    * Restarting with reloader

This is useful when you want to see changes without having to rebuild the whole suite.

To build the static website:

    $ ./site-tool build

Generated HTML files and assets will go to the `./build/` directory.

To deploy the website to AWS (caveat: You'll need to set some environment variables for secrets)):

    $ ./site-tool deploy

There's also a command for creating new posts:

    $ ./site-tool post blog --title="My title"
    Created pages/blog/2016/my-title.md
    $ cat pages/blog/2016/my-title.md
    title: My title
    date: 2016-10-05
    published: false

License
-------

Anything here is released under the terms of the [Do What The Fuck You Want To Public License](http://sam.zoy.org/wtfpl/).
