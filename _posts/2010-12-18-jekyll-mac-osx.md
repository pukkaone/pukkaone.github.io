---
layout: post
title: Jekyll on Mac OS X 10.6
---

I'm using [Jekyll](http://github.com/mojombo/jekyll) to render this blog.  I
want to proof the look of the posts before pushing them to my
[GitHub Pages](http://pages.github.com/) site, so I installed Jekyll on my
MacBook Pro.  The following steps install packages into the Ruby and Python
distributions provided with Mac OS X 10.6.


### Upgrade RubyGems

The [liquid](http://www.liquidmarkup.org/) gem requires RubyGems 1.3.7 or
higher.  Download the
[newest RubyGems version](http://rubyforge.org/frs/?group_id=126), untar the
archive, then run

    sudo ruby setup.rb


### Install Jekyll

    sudo gem install jekyll


### Install RDiscount

I want to use [RDiscount](http://github.com/rtomayko/rdiscount/) instead of
[Maruku](http://maruku.rubyforge.org/) to render
[markdown](http://daringfireball.net/projects/markdown/).

    sudo gem install rdiscount

Configure the markdown processor by adding this line in the `_config.yml` file:

    markdown: rdiscount


### Install Pygments

I'm going to write about code, so I installed the
[Pygments](http://pygments.org/) syntax highlighter.

    sudo easy_install Pygments
