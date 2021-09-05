---
title : "블로그 제작"
layout : archive
permalink : categories/blog_dev 
author_profile : true
sidebar_main : true
---

{% assign posts = site.categories['Blog Dev'] %}

{% for post in posts %}
    {% include archive-single.html type = page.entries_layout %}
{% endfor %}
