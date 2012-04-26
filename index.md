---
layout: page
title: Ratazzi's Blog
---
{% include JB/setup %}

因为 GAE 种种原因，放弃了，转而使用现在这种基于 Jekyll 引擎的方式，Git + Markdown = Blog ，旧的博客地址依然可用，不打算迁移了，因为现在看来之前的内容确实没有太多价值，部分文章我会慢慢整理后迁移过来。

非常喜欢现在这种简单的方式，用最喜欢的编辑器，摆脱传统的浏览器写作然后 POST 的方式。

* [http://github.com/ratazzi](http://github.com/ratazzi)
* [http://twitter.com/ratazzi_potts](http://twitter.com/ratazzi_potts)

## 最近的文章
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
