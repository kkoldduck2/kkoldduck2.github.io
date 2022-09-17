---
title: "accepted"
layout: archive
permalink: categories/accepted
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.accepted %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}