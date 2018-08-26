---
layout: post
title: "Projects Blog"
author: "Predrag Stošić"
photo_url: "/assets/projects-blog/projects-blog.png"
---

While looking at my [github profile](https://github.com/stoleS), I thought it would be good to better describe some more important projects in a blog type website. Since I worked with [Jekyll](https://jekyllrb.com/) before, I decided to use it as a backbone for this project. So here it is, in all of it's glory. In the next few paragraphs I'll try to explain some of more important things of this project.

### Dependencies

For this project I used only one npm dependency, Bulma:

- [Bulma](https://bulma.io/)

You can install it like this:

```sh
npm install --save bulma
```

### Folder structure

Here is the folder structure for this jekyll blog:

- `_config.yml` : Jekyll [configuration](http://jekyllrb.com/docs/configuration/)
- `_layouts/`
  - `default.html` : default template
  - `post.html` : post template
- `_includes/`
  - `nav.html` : navigation bar
  - `footer.html` : header and footer used by the default template
  - `head.html` : site navigation
  - `hero` : page hero image and text
- `_posts` : posts for this blog
- `_sass`: sass stylesheets
- `assets` : assets for the blog and posts
- `index.html` : main page
- `archive.html` : archive page
- `contact.html` : contact page
- `package.json` : npm dependencies

## Layouts

As you can see from the folder structure of this blog, we have only two layouts:

1. default.html
2. post.html

### 1) default.html

This layout is used as a template for all the pages on this blog, except for single post page, and it looks like this:
{% raw %}

```html
{% include head.html %}
{% include nav.html %}
{% include hero.html %}
<div  class="container">
  <section  class="articles">
    <div  class="column is-8 is-offset-2">
      {{ content }}
    </div>
  </section>
</div>
{% include footer.html %}
```

{% endraw %}

First of all, we need to include blocks that every page will use, starting with head, then navigation and finishing with hero section. After that we have container that holds all of our articles or only one of them if we are reading a specific one. And at the end we include our footer.

### 2) post.html

This is a single post layout and it extends default layout. It is used to hold a single post on it's rendered page. Some of the more interesting parts of post page are:

- Assumed read time
- Next/previous post

#### Assumed read time

![Read Time](/assets/projects-blog/read-time.png)

Read time is calculated like this:
{% raw %}

```
{% assign words = page.content  | number_of_words %}
{% if words < 270 %}
  1 min read
{% else %}
  {{ words | divided_by:135 }} min read
{% endif %}
```

{% endraw %}

So, the first thing we will want to do is get the word count.
{% raw %}

```
{% assign words = page.content  | number_of_words %}
```

{% endraw %}

After that, we divide it with number of words you can read per minute. Average is 180, but that's quite slow, so we bump it up to 270. If post doesn't contain more than 270 words, we'll say that the read time is one minute. But if a post contains more than 270 but less than 540, Liquid will round that down to one minute, so we have to check if it is less than 540. Better explanation, and original solution is available by [Carlos Alexandro Becker](https://carlosbecker.com/posts/jekyll-reading-time-without-plugins/) in his blog post.

#### Next/previous post

We can easily access next and previous post from one that we are currently looking at with page.next and page.previous:
{% raw %}

```
{% if page.next.url %}
  <a  href="{{ page.next.url }}">{{ page.next.title }}</a>
{% endif %}
{% if page.previous.url %}
  <a  href="{{ page.previous.url }}">{{ page.previous.title }}</a>
{% endif %}
```

{% endraw %}

## Index page pagination

To implement the pagination on this blog's index page, I'm using Jekyll's [pagination](https://github.com/sverrirs/jekyll-paginate-v2) plugin:
{% raw %}

```
{% if paginator.total_pages > 1 %}
  <nav role="navigation" aria-label="pagination">
  {% if paginator.previous_page and paginator.next_page != true %}
    <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">&laquo; Previous</a>
    <a disabled>Next page &raquo;</a>
  {% endif %}
  {% if paginator.page_trail %}
    <ul>
    {% for trail in paginator.page_trail %}
      <li>
        <a href="{{ trail.path | prepend: site.baseurl }}"  title="{{trail.title}}"  class="{% if page.url == trail.path %}is-current{% endif %}">{{ trail.num }}</a>
      </li>
    {% endfor %}
    </ul>
  {% endif %}
  {% if paginator.next_page and paginator.previous_page != true %}
    <a  href="{{ paginator.next_page_path | prepend: site.baseurl | replace: '//', '/' }}">Next page &raquo;</a>
    <a disabled>&laquo; Previous</a>
  {% endif %}
  </nav>
{% endif %}
```

{% endraw %}

First of all, we check if there are more than one page in our paginator (number of posts per page is defined in yaml front matter in paginator plugin section):
{% raw %}

```
{% if paginator.total_pages > 1 %}
```

{% endraw %}
After that, we have pagination container:
{% raw %}

```
<nav role="navigation" aria-label="pagination">
```

{% endraw %}
Next, there is a check for previous and next page. If there is a previous, but there isn't a next page, we display previous page button with path to that page, and disabled "next page" button:
{% raw %}

```
{% if paginator.previous_page and paginator.next_page != true %}
  <a href="{{ paginator.previous_page_path | prepend: site.baseurl | replace: '//', '/' }}">&laquo; Previous</a>
  <a disabled>Next page &raquo;</a>
{% endif %}
```

{% endraw %}
Between "next" and "previous" buttons there are page number buttons:
{% raw %}

```
{% if paginator.page_trail %}
  <ul>
  {% for trail in paginator.page_trail %}
    <li>
      <a href="{{ trail.path | prepend: site.baseurl }}"  title="{{trail.title}}"  class="{% if page.url == trail.path %}is-current{% endif %}">{{ trail.num }}</a>
    </li>
  {% endfor %}
  </ul>
{% endif %}
```

{% endraw %}
Here we check if there are multiple pages, and if there is, we can display their number with link to access each one of them.

## Archive page

Archive page is one of the more important pages in every blog. It's purpose is to give as easy access to all of the posts in a blog, sorted by date. Building the archive page in Jekyll is quite easy:
{% raw %}

```
{% for post in site.posts %}
  {% assign currentDate = post.date  | date: "%B %Y" %}
  {% if currentDate != myDate %}
    {% unless forloop.first %}
      </ul>
    {% endunless %}
    <h1>{{ currentDate }}</h1>
    <ul>
    {% assign myDate = currentDate %}
  {% endif %}
  <li>
    <a href="{{ post.url }}">
      <span>{{ post.date  | date: "%B %-d, %Y" }}</span> - {{ post.title }}
    </a>
  {% if forloop.last %}
    </ul>
  {% endif %}
{% endfor %}
```

{% endraw %}

We loop through all blog posts, grab the post date (month and year), print that first, then compare dates of the post and loop date, and after that print the posts which are in current date range.

### Conclusion

This was a very interesting and fun project and I gained a lot of knowledge on how templating and static site generator works. Jekyll is a great tool for building simple blogs, and even some more complex pages. So if you are in a need for such a website, you shoud check it. GitHub repository for this project is available [here](https://github.com/stoleS/projects-blog-source). Thanks for reading and happy coding.
