---
title: "Database"
layout: archive
permalink: categories/database
author_profile: true
sidebar_main: true
---

<!-- ������ ���ԵǾ� �ִ� ī�װ� �̸��� ��� site.categories['a b c'] �̷�������! -->

***

{% assign posts = site.categories.Collage%}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}