---
layout: post
title: "How to use Emoji's in GitHub Pages/Jekyll Blogs"
date: 2017-06-19
---

I setup my blog using GitHub pages and of course my first priority was to figure
out how to include Emoji's in my blog posts.

At first I took the naive route and created a post in included an Emoji tag such
as :smile: in my markdown file expecting it to work. I was disappointed!

A quick google turns up this simple and easy to use Jekyll plugin called
[Jemoji](https://github.com/jekyll/jemoji).

I added the following line to my `Gemfile`

```
gem 'jemoji'
```

and added the following to my `_config.yml` file

```
gems:
  - jemoji
```

Now in any page I can use emoji as I normally would :collision::star::smiley:
:dizzy::ok_hand::collision::star::smiley::dizzy:

Cool!
