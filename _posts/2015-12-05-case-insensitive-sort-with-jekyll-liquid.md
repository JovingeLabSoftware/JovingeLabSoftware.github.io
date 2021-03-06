---
layout: post
category: blog
date: "2015-12-05 09:43 -0500"
author: Eric
tags: 
  - code snippet
  - jekyll
  - liquid
published: true
title: Case insensitive sort with Jekyll / Liquid
---





The Liquid template language is fairly restricted in order to ensure security when generating static web pages from templates, making use of predefined filters and the expense of user defined functions.

However, there does not seem to presently be a case-insensitive sort filter (It does seem to be [in the works](https://github.com/Shopify/liquid/pull/554)--but `sort_natural` does not seem to be supported yet in the version of jekyll on github--2.4, nor on 3.0.1 which I also downloaded and tried.)

Here is a way to achieve case insensitive sort by using `downcase` prior to sorting and comparing.  In this example we are creating an index of posts based on their tags.  The drawback to this approach is that then we can no longer use `site.tags[tag]` to access the list of posts for each tag (because the case of `tag` may have changed).  Instead, we have to spin through the entire list of posts for each tag to pull out the matching posts.

Finally, note that we create a comma delimited list out of the tags and then split this list into an array.  Care must be taken to avoid **extra whitespace**, so I cram the capture statement togeher all on one line.  (Later versions of Jekyll introduce `strip` which is handy for getting rid of leading and trailing whitespace but github uses Jekyll 2.4 as of 12/2015)

In addition to being a stop-gap solution for case insensitive sort until `sort_natural` goes live, this snippet demonstrates many of the fundamentals of Liquid templating.
 
{% raw %}
```Liquid
---
layout: default
title:  Tags
---

<h2> Index of posts by Tag:</h2>
<section>
  
	{% capture tags %}{% for tag in site.tags %}{{tag[0]}}{{','}}{% endfor %}{% endcapture %}
	{% assign sortedtags = tags | downcase | split:"," | sort %}
	{% for tag in sortedtags %}
	  <b><a name="{{ tag | downcase }}"> </a>{{ tag | capitalize }}</b>
	    <ul>
	        {% for post in site.posts %}
	          {% for t in post.tags %}
	             {% capture tdown %}{{ t | downcase }}{% endcapture %}
	          	 {% if tdown == tag %}
  			       <li><a href="{{ post.url }}">{{ post.title }}</a></li>
				 {% endif %}
	          {% endfor %}
		    {% endfor %}
		</ul>
	{% endfor %}
</section>
```
{% endraw %}
