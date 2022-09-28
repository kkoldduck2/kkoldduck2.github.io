---
title: "worklog"
layout: archive
permalink: categories/worklog
author_profile: true
sidebar_main: true
---

{% assign posts = site.categories.worklog %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
