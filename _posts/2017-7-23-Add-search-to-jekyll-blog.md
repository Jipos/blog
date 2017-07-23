---
layout: post
title: Search all posts in Jekyll blog
---

I want to be able to search the posts on my Jekyll blog.

The following page got me started:
> https://learn.cloudcannon.com/jekyll/jekyll-search-using-lunr-js/

The actual documentation of the jekyll-lunr-js-search plugin, used in the above page can be found here:
> https://github.com/slashdotdash/jekyll-lunr-js-search

Some things I ran into:
* adding a search.html to the root of my repo, resulted in a /blog/search/ page getting created.
* the jekyll-lunr-js-search plugin adds the search related JS files to the js folder in the root of the repo. This folder is served as /blog/js/. So in order to access the js search.js file in the browser, I need to call "../js/search.js" instead of "js/search.js"

**TIP:**
After running 'jekyll serve', a _site folder is created which contains the site as it is served. This can help you solve problems like being unable to access a JS file.
