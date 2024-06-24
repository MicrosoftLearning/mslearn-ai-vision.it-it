---
title: Esercizi relativi a Visione di Azure AI
permalink: index.html
layout: home
---

# Esercizi relativi a Visione di Azure AI

Gli esercizi seguenti sono progettati per supportare i moduli in Microsoft Learn.


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %}

{% for activity in labs  %} {% if activity.lab.title contains "Azure AI Custom Vision" %}  
    {% continue %}  
  {% endif %} 
  - [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
