---
layout: post
title:  "test2 for page.html"
date:   2017-03-05 17:50:00
comments: true
categories: main
---

img embedding using kramdown style
![My helpful screenshot]({{ site.url }}/assets/images/fig5.png){:class="img-responsive"}

img embedding using html style code.
<img src="{{ site.url }}/assets/images/fig5.png">

download pdf file.

[Get the PDF]({{ site.url }}/assets/pdfs/test.pdf)

kramdown style. 
![Get the PDF]({{ site.url }}/assets/pdfs/test.pdf)

from stackoverflow
[download link][1]

download a file
[Get the file]({{ site.url }}/assets/file/gp.pyc)

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
[naver]: http://www.naver.com/1
[1]: {{site.url}}/assets/pdfs/test.pdf
