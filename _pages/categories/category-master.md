---
title: "master"
layout: archive
permalink: categories/master
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.master %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}