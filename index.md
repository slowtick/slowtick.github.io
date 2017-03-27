---
layout: default
title: personal pages
description: personal blog and articles about web services & related technologies written by slowtick
jsonld_include: slowtick.json
---

### Pages

{% assign all_listed_pages = site.html_pages | sort: 'index_list_at_position' %}
{% for a_page in all_listed_pages %}
  {% if a_page.index_list_at_position %}
- [{{ a_page.index_list_link_text }}]({{ a_page.url }})
  {% endif %}
{% endfor %}

<script type="application/ld+json">
{
  "@context":"http://schema.org",
  "@type":"ItemList",
  "itemListElement":[ {% assign all_listed_pages = site.html_pages | sort: 'index_list_at_position' %} {% for a_page in all_listed_pages %} {% if a_page.index_list_at_position %}
    {
      "@type":"ListItem",
      "position":{{a_page.index_list_at_position}},
      "url":"{{ a_page.url | prepend: site.github.url }}"
    } {% if forloop.last == false %} , {% endif %} {% endif %} {% endfor %}
  ]
}
</script>