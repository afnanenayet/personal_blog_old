# [afnan.io](http://afnan.io)
[![Build Status](https://travis-ci.org/afnanenayet/personal_blog.svg?branch=master)](https://travis-ci.org/afnanenayet/personal_blog)

This is the readme for afnan.io. It is built with Travis CI and verified with
HTML proofer. Comments are powered by Disqus.

I am using the [Kiko Plus](https://github.com/AWEEKJ/Kiko-plus)
Jekyll theme.

# Rakefile usage

```shell
# Create new post
$ rake post title="A Title" [date="2015-08-16"] [tags="[tag1, tag2]"]

# Create new draft post
$ rake draft title="A Title" [date="2015-08-16"] [tags="[tag1, tag2]"]

# Install Jekyll Plugins. Do before running in local.
$ rake geminstall

# Run in Local
$ rake preview
```
