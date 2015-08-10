---
layout: post
title:  "Host your blog for nothing"
subtitle: "Blog hosting with GitHub"
date:   2015-08-10
categories: cloud, github
tags: cloud ruby jekyll github
author: Brett Cherrington
---

Last year I wrote a blog post (see below) about hosting your blog for pennies. Well why would
you do that when you can host your blog for FREE on GitHub with GitHub pages.

There's a great [guide from GitHub themeselves here](https://pages.github.com/) so there's no
need for me to repeat what they say. But if you followed the previous guide and have built your blog using Jekyll already then moving it across to GitHub pages is really easy. Essentially you just create a new GitHub repo called *username*.github.io where *username* is your GitHub username. Then check in your Jekyll website, GitHub will do the rest and your site will be instantly available on http://*username*.github.io

Something to watch out for though is to make sure you __don't__ check in the _site folder as this will be generated for you by the GitHub Jekyll process. If you do check in this folder you'll see that your changes are not reflected on the site and it'll appear static.

Also note that can even have the site hosted under your own domain name. Again there's a [great guide here](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/). All you need to do is create a simple CNAME file in your repo and re-point your domain to the appropriate *username*.github.io domain within your regstrars DNS control panel.

Oh, did I mention, you get source control of your website as well!!