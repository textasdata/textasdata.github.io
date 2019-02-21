---
layout: resources
title: Tutorials
permlink: /resources/tutorials_main
---

 {% for page in site.pages %} {% if page.url contains "tutorials/" %}

{{ page.title }}

{{ page.description }}

<div class="col-xs-6">
                <div class="thumbnail">
                    <img src="{{ site.baseurl }}/images/post_images/{{ page.thumnail }}" alt="Sample Image">
                    <div class="caption">
                        <h3>{{ page.title }}</h3>
                        <p>{{ page.description }}</p>
                        <p><a href="{{ site.baseurl }}{{ page.url }}" class="btn btn-primary">View tutorials</a></p>
                    </div>
                </div>
            </div>
            {% endif %} {% endfor %}