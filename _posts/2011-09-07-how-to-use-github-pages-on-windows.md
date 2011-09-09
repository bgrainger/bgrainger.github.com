---
layout: post
title: How to use GitHub Pages on Windows
---

## Install

1. Install [RubyInstaller](http://rubyinstaller.org/). (I used version 1.9.2-p290.)
2. Install the RubyInstaller [DevKit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit) (be sure to customise the install path, e.g., to `C:\RubyDevKit`).
3. Install [Jekyll](https://github.com/mojombo/jekyll/wiki/install): Run "Start Command Prompt with Ruby", then `gem install jekyll`
4. Install [Python](http://python.org/download/). (I used Python 2.7.2 Windows Installer.)
5. Install Python [setuptools](http://pypi.python.org/pypi/setuptools). (I used setuptools-0.6c11.win32-py2.7.)
6. Add `C:\Python27\Scripts` to your path.
7. Install [pygments](http://pygments.org/): `easy_install pygments`
8. Apply the [patch](https://gist.github.com/1185645) (linked [here](http://madhur.github.com/blog/2011/09/01/runningjekyllwindows.html)) to prevent "Liquid error: bad file descriptor" being output when you use pygments: `cd /c/Ruby192/lib/ruby/gems/1.9.1/gems/albino-1.3.3/lib; patch < 0001-albino-windows-refactor.patch`

## Set up the GitHub Pages Repository

1. Install [msysGit](http://code.google.com/p/msysgit/).
2. Create a new repo as per the [GitHub Pages documentation](http://pages.github.com/). 
    * An easy way to get started is to copy or fork [an existing repo](https://github.com/mojombo/mojombo.github.com).
3. Write and save a post in the `_posts` folder.

## Test

1. Run "Start Command Prompt with Ruby" and change to your pages repo's working copy.
2. Enter `jekyll --auto --server` to build your site and launch a web server that rebuilds your site every time you save in your editor.
3. Browse to [http://localhost:4000](http://localhost:4000) to see your site.

## Publish

1. Create a GitHub repository named after your user name (e.g., `bgrainger.github.com`).
2. Push your local repository to GitHub.

## Further Reading

* [Running Jekyll on Windows](http://madhur.github.com/blog/2011/09/01/runningjekyllwindows.html)
* [Moved Blog to Jekyll and GitHub Pages](http://www.alexrothenberg.com/2011/01/27/moved-blog-to-jekyll-and-github-pages.html)

