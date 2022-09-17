---
title: "fail"
layout: archive
permalink: categories/fail
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.fail %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}