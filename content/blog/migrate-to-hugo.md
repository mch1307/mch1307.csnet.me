+++
author = "mch1307"
categories = []
tags = ["hugo", "netlify"]
date = "2018-03-18"
description = "blog.csnet.me now powered by Hugo, deployed and hosted on Netlify"
featured = "migrate-to-hugo.png"
featuredpath = "date"
linktitle = ""
title = "From WP.com to Hugo on Netlify"
type = "post"
archives = ["2018"]
+++

When I decided to start blogging and have my own "site", I chose Wordpress.com as it is was for me the easiest and most comfortable solution.
I have been using it for a few month and to be fair, I think the product and service are ok. 

The main pain points to me are:

* authoring and preview
* having to move to a more expensive plan in order to install plugin

So I decided to have a look at [Hugo][1], a fast static web site generator.

The most attracting points with Hugo were:

* MD files editing with my favorite editor (vscode)
* Embedded local "preview" server
* Save and version my files to GitHub.com (+ delivery pipeline)
* Ability to customize and implement features, even if more difficult than installing a wp plugin

The only left point was how to host my site, and that's where [Netlify][2] enters the game.

[Netlify][2] allows you to automatically build, deploy and run your Hugo site upon commiting change in GitHub.

It is easy to setup as described in the [Hugo documentation][3].

Netlify offers classic features like SSL or DNS, but also very nice ones such as global CDN.




[1]: http://gohugo.io
[2]: http://netlify.com
[3]: http://gohugo.io/hosting-and-deployment/hosting-on-netlify/
