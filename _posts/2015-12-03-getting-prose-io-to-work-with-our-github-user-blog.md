---
layout: post
category: blog
tags: null
published: false
title: Getting prose.io to work with our github user blog
---

I have had some trouble getting prose.io to play nicely with our user blog, but after studying the [priose.io starter repository](https://github.com/prose/starter) I think I may have it working now (see `_config.yml` below).  In addition to editing `_config.yml` as below, I also added [links.jsonp](https://github.com/JovingeLabSoftware/JovingeLabSoftware.github.io/blob/master/links.jsonp) to root directory of repository.

One trick: to actually publish a new post you need to:
1. Click new file in the \_posts directory
2. Edit your post
3. Click save
4. **Refresh the page** to get the "unpublished" button to appear in the formatting toolbar.
5. Click "unpublished" to toggle it to "published"
6. Save again

Here is what the prose section of `_config.yml` looks like currently:

```
prose:
  rooturl: '_posts'
  siteurl: 'http://jovingelabsoftware.github.io'
  relativeLinks: 'http://jovingelabsoftware.github.io/links.jsonp'
  media: 'media'
  metadata:
    _posts:
      - name: "category"
        field:
          element: "hidden"
          value: "blog"
      - name: "layout"
        field:
          element: "hidden"
          value: "post"
      - name: "title"
        field:
          element: "text"
          label: "Title"
          value: ""
      - name: "tags"
        field:
          element: "multiselect"
          label: "Add Tags"
          placeholder: "Choose Tags"
          options:
            - name: "R"
              value: "R"
            - name: "Couchbase"
              value: "couchbase"
            - name: "Node.js"
              value: "node.js"
            - name: "Shell"
              value: "shell"
            - name: "Code Snippet"
              value: "code snippet"
            - name: "Shiny"
              value: "shiny"
            - name: "ggplot2"
              value: "ggplot2"
            - name: "LINCS"
              value: "lincs"
    _posts/static:
      - name: "layout"
        field:
          element: "hidden"
          value: "page"
      - name: "title"
        field:
          element: "text"
          label: "Title"
          value: ""
      - name: "permalink"
        field:
          element: "text"
          label: "Permalink"
          value: ""
```

