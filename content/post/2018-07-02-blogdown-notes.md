---
title: Blogdown notes
author: Xijin Ge
date: '2018-07-02'
slug: blogdown-notes
categories:
  - R
tags:
  - Blogdown
header:
  caption: ''
  image: ''
---
This website is created using RStudio following this [tutorial](https://alison.rbind.io/post/up-and-running-with-blogdown/) with tips from [Ming Tang](https://divingintogeneticsandgenomics.rbind.io/post/hugo-academic-theme-blog-down-deployment-some-details/).

To start a new post, choose "New post" from Addins.
Use Markdown for texts.  Use RMarkdown if it contains R markdown to render plots, etc. 

Choose which files you want to work on from the right bottom window. Save the files changed and the webpage is rendered automatically in the Viewer window.


To disable a widget, change the header from active=true to active=false

Use Git GUI to push to GitHub. 

Just like magic, the website will refresh automatically at Netlify.com. 

To add a lot of figures to a [post](https://gex.netlify.com/post/using-amazon-ec2-to-run-large-data-analysis-cheaply/), creaste sub-folders like postEC2, and then in the Markdown file use: 
```
 ![image](/img/postEC2/image025.png)
```
For R markdown files, the current folder has to be blog/content/post.  For complex calculations, such as the iDEP R Markdown, try the markdown separately, then import to Blogdown.