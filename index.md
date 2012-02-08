---
layout: page
title: 我的新博客
---
{% include JB/setup %}

因为 GAE 恶心的收费方式，放弃了，本来打算在原来的基础上改写的，现在有了更好的解决方案，不过在主题和文章迁移完成之前
暂时还得先用这个域名，这种 git + markdown = blog 的方案真是太爽了，push 后自动生成静态页面。

## TODO
* 主题的制作，因为使静态页面，可能需要有比较大的变动
* 文章的迁移，一些没有价值的废话可能会去掉

## 最近的文章
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
