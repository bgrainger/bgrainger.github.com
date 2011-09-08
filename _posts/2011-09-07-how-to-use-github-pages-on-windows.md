---
layout: post
title: How to use GitHub Pages on Windows
---

## Install

1. Install [RubyInstaller](http://rubyinstaller.org/). (I used version 1.9.2-p290.)
2. Install the RubyInstaller [DevKit](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit) (be sure to customise the install path, e.g., to `C:\RubyDevKit`).
3. Install [Jekyll](https://github.com/mojombo/jekyll/wiki/install): `gem install jekyll`

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
