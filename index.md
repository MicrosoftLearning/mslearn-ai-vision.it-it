---
title: Esercizi di Visione di Azure AI
permalink: index.html
layout: home
---

# Esercizi di Visione di Azure AI

Gli esercizi seguenti sono progettati per supportare i moduli in Microsoft Learn.


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% for activity in labs  %} 
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
