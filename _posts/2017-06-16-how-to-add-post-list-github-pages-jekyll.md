---
layout: post
title: "How to display a list of Posts using Github Pages & Jekyll"
date: 2017-06-16
---

I recently created this site using Github Pages for hosting and Jekyll for
static website generation. I followed [this guide](https://help.github.com/articles/creating-a-github-pages-site-with-the-jekyll-theme-chooser/) initially for the basic setup. It left me with a basic page but I wanted to
have a home page that showed a list of all my posts in descending chronological
order like the screenshot below:
![how i wanted my page to look]({{ site.url }}/assets/1/desired-layout.png)

I'm using the [Minimal theme](https://github.com/pages-themes/minimal) but this
should work for the other themes too.

## Creating a Post layout

The first thing we need to do is create a custom layout to display each of our
blog posts.

1. Create a new folder in your project root directory called `_layouts`

```bash
$ mkdir _layouts
```

2. Change to the `_layouts` directory and create a file called `post.html`

```bash
$ cd _layouts && touch post.html
```

3. Open `post.html` in your preferred text editor and insert the following HTML:

```html
{% raw %}
---
layout: default
---
<h1>{{ page.title }}</h1>
<p class="meta">{{ page.date | date_to_string }}</p>

<div class="post">
  {{ content }}
</div>
{% endraw %}
```

Let me explain what the above HTML markup does. The first three lines are what
Jekyll calls [YAML front matter](https://jekyllrb.com/docs/frontmatter/). It
indicates that this HTML should be inserted into the default layout i.e the
default layout that comes out of the box with your chosen theme. Next we are
displaying the title of the post in a header tag, the date and the post content.

Now let's create some posts!

## Creating Posts

We need to create some actual blog posts to display in our list.

1. Create a new folder in your project root directory called `_posts` just like
you did for `_layouts`

```bash
$ mkdir _posts
```

2. Change to the `_posts` directory and create a file in the format
`YEAR-MONTH-DAY-your-post-title.md` for example `2017-06-10-my-first-post.md`.
It is very important to stick to this format if you want Jekyll to generate
your posts for you!

```bash
$ touch 2017-06-10-my-first-post.md
```

3. Open `2017-06-10-my-first-post.md` or whatever you called your file in your
preferred text editor and insert the following text:

```
{% raw %}
---
layout: post
title: "First post!"
date: 2017-06-16
---

First post!
{% endraw %}
```

What this markdown does should be fairly self explanatory so I won't bother
explaining.

Repeat these steps and create another post file with a different date and title
so you can see the list in action later.

## Creating a Posts list layout

Now we need to create the layout that will display a list of Posts.

1. Create a new file in your project root directory called `index.html`

```bash
$ touch index.html
```

2. Open `index.html` in your preferred text editor. Copy in the following HTML:

```html
{% raw %}
---
layout: default
---
{% for post in site.posts %}
<article>
<a href="{{ post.url }}"><h2>{{ post.title }}</h2></a>
<p>{{ post.date | date_to_string }}</p>
<div>
  {{ post.content }}
</div>
<article>
{% endfor %}
{% endraw %}
```

In this file we are injecting this layout into the default layout as before.
Then we use a Jekyll tag denoted using `{{ }}` and creating a `for` loop.
Jekyll is blog aware so we get objects such as `site` and properties such as
`posts`. We create an `<article>` tag and show a header, post date and content
of the post itself. Finally we end the `for` loop using another Jekyll tag.

Finally, push your changes to GitHub and check out your page

1. Add all your modified and newly created files

```bash
$ git add .
```

2. Commit your changes

```bash
$ git commit -m "made an awesome list of posts"
```

3. Push to the remote repository

```bash
$ git push origin master
```

[Send me a tweet](https://twitter.com/danielsouthlc) if you have any feedback or
found this post useful :smile:
