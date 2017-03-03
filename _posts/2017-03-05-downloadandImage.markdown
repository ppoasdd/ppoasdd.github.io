---
layout: default
title:  "test2 for page.html"
date:   2017-03-05 17:50:00
comments: true
categories: main
---

Go to [naver][naver]

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
including Image 
[My helpful screenshot]({{ site.url }}/assets/images/fig5.png)

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


{% include disqus.html %}

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
[naver]: http://www.naver.com/1
[1]: {{site.url}}/assets/pdfs/test.pdf
