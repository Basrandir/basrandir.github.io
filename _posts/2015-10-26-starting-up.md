---
layout: post
title: Starting up with Jekyll and Github
category: blog
tags: web, jekyll, github
---
I've decided to actually create a website in order to document my learning as well as interesting information or methodology I stumble across. Essentially a way for me to keep track of certain details that I might need to refer back to. Maybe it'll be helpful for others too.

Up to this point I've used a self-hosted wiki to manage this type of information (as well as cooking recipes). This blog is meant to be essentially an extension of that with more detail and my thought process and reasoning behind the decisions I make. Over time I'll probably transfer some information from the wiki over to here in a nicer format.

## Journey to Jekyll
Static site generators had always interested me. I'd had enough experience with overwrought dynamic CMSs to be a little tired of them. I really don't see the need for them for my small little website. I also generally like to be as close as possible to the actual code as I can. Abstractions on top of abstractions get a little tedious for me.

This isn't my first time experimenting with static site generators. I actually built one myself for a client's website a few years ago. It was more of a programming project than an attempt at a viable, commercial generator. I used Python for the actual generator while making use of the Jinja2 template engine. Honestly I was pretty proud of it. Especially for what amounted to a weekends worth of work.

A few days ago when I actually decided to set up this website I considered using my own generator for this site. But I quickly realized that it would need quite a bit of work to function as an actual blog generator. And among the publicly available generators, Jekyll was used automatically by Github Pages (which this site is hosted on). Hence my decision. It doesn't hurt that Jekyll is the most popular and so there's a lot of resources available for it. Finally, I have *zero* experience with Ruby so it might be a nice learning tool if I ever end up going into the nitty-gritty of Jekyll.

## Setting up Github Pages and initializing Jekyll
Creating a Github site for myself was fairly straightforward. A simple repository with a *CNAME Record* file that let's Github know my custom domain. And finally a few *A Records* with my domain pointed to Github. It's incredibly simple and straightforward.

Setting up Ruby on my computer was an interesting experience. I recently got a Macbook Pro which would be my first experience with OS X since a Grade 10 Communications course. So I figured I'd set it up first on the Macbook rather than my Linux box. The first thing everyone seems to recommend is to install Homebrew, the presumably best package manager on OS X. From there it's on to setting up Ruby.

There's a lot of different ways of setting up Ruby installs it seems. My version of OS X came with Ruby 2.0.0 pre-installed but I wanted the latest stable which, at the time of writing, was 2.2.3. There's a multitude of ways to accomplish this but I settled on using ruby-install for the actual installation and chruby to manage the various Ruby installs.

The next step was configuring my Ruby environment. Bundler is helpful in this because it ensures all the Ruby gems match the correct version needed. All I had to do was create a simply *Gemfile* file with the names of all the gems I wanted (and their specific versions if needed). In my case the only gem I needed was Jekyll.

I forwent using the github-pages gem in favour of setting up my Jekyll system myself. After Bundler installed Jekyll, all I had to do was run `jekyll new .` in my repository. Jekyll then built the entire framework which I haven't touched at all so far. The only change will be this post.

## To Infinity
Going forward I'm going to keep this website incredibly simple. There's going to be a design change from this default Jekyll skin but I doubt I'll be doing anything truly fancy. I generally prefer a minimalistic style but we'll see if I'll build something a little more complex than Richard Stallman's website.

Hopefully this'll be fun.

Or at least interesting.
