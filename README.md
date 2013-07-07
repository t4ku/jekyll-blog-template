Jekyll Blog Template
====================

This is a jekyll's blog template I use for my blog.
The following is a list of additions and plugins to jekyll's default template.

* [compass](https://github.com/chriseppstein/compass/wiki/Jekyll-Integration)
* [jekyll-prism-plugin](https://github.com/gmurphey/jekyll-prism-plugin)

Setup
-----

```
git clone https://github.com/t4ku/jekyll-blog-template.git
cd jekyll-blog-template
git submodule init
```

Usage
-----

### generate site & preview 

```
jekyll serve
```

Deploying to Heroku
-------------------

```
heroku config:add BUILD_PACK=https://github.com/t4ku/heroku-buildpack-ruby-jekyll.git
```

