---
layout: post
title: "How I made this"
category: posts
---

# Setting up a website with Google App Engine, Travis CI, and Jekyll

Recently, I decided that I wanted to change the look of my website. I'm 
not really a web developer so I wanted a simple setup. I like Markdown, 
especially the way Github renders markdown documentation. 

Feel free to modify/check out the [source](https://github.com/afnanenayet/personal_blog) 
for this website.

## Setup

Setting up Jekyll is pretty simple - it's a Ruby gem so you have to 
install Ruby. I have Homebrew, so it was as simple as 

    brew install ruby

    sudo apt install ruby

And from there, I installed Jekyll as a gem.

    gem install jekyll

You'll probably also want the [Travis CI command line tools](https://github.com/travis-ci/travis.rb)

## Using Jekyll

Jekyll is a very easy client to use. It takes markdown and converts it to HTML. 
The websites are static, making them fast as a nice bonus. Jekyll is primarily 
designed for written content like this - blogs. 

Your initial steps are outlined in the Jekyll 
[quick start guide](https://jekyllrb.com/docs/quickstart/)

You have two main types of pages: blog posts, and pages.

Jekyll has a config file, called `_config.yml` where you put in site wide 
settings and data. You will enter your site's URL, data that might be relevant 
to the particular theme you're using, your site's description, etc.

Pages are just what they seem like - pages. To create one, you make a markdown 
file, and you can stick it in the root of your project. Let's pretend it's 
called page.md.

At the beginning of every markdown file that gets converted to an HTML page,
you have to specify some metatdata. Some example markdown for a post would 
look like 

```
---
layout: post
title: "This is a post!"
category: posts
---
# This is a heading
Here is some content
```

These layouts are converted to HTML by the layouts specified in the `_layouts`
directory. For example, if you specify that you're using the `post` layout, 
Jekyll will use `_layouts/post.html` as the template. 
You can use a Jekyll theme, or create your own layouts. Jekyll 
also allows you to use Sass to create your custom layouts. If you're using a 
Jekyll theme, any files you put in the `_layouts` folder will override the 
theme. 

Take a look at the [Jekyll documentation](https://jekyllrb.com/docs/templates/)
for more details.

## Setting up Travis CI

Once you have a Jekyll blog and you can run it locally, that's great - but how 
are you going to deploy it? One option is to use [Github pages](https://pages.github.com), which are 
powered by Jekyll themselves. This is easy, but doesn't offer as much flexibility 
as Google App Engine.

Travis CI offers continuous integration, a way to build, test, and deploy 
our compiled website to Google App Engine. Travis CI has documentation 
on how to deploy to Google App Engine by adding a few lines to your 
`.travis.yml`. Check out my source code for an example `.travis.yml` file. 
Take a look at the Travis documention on setting up a [Ruby](https://docs.travis-ci.com/user/languages/ruby/) 
environment and for [deploying to GAE](https://docs.travis-ci.com/user/deployment/google-app-engine/)

If you're using Travis to deploy to GAE, I recommend that you use [jekyll-app-engine](https://github.com/jamesramsay/jekyll-app-engine)
to generate your app handlers. 

Once you have your `.travis.yml` config set up, go to travis-ci.org and turn 
on continuous integration on the repo you're using for the website. 

## Deploying to Google App Engine

Create an Google cloud account. The free tier allows you to have an app engine 
instance with 5 gigs of storage. The Travis documentation shows you how to 
retrieve an access key in JSON format and securely encrypt it for use in the 
Travis docker image when it tries to upload your files to GAE. Make sure you're 
using the right URL in your Jekyll configuration, otherwise when your 
website is live, it won't use the right URL to load assets. 

If your configurations are correct, then all the markdown files you commit 
to your repo should be built by Travis and reflected by your live site 
on Google App Engine.


