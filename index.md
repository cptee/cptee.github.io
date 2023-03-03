---
layout: default
title: Cptee's Cybersec Notes
description: "Hi! My name is Lucas and I go as Cptee on my hacking endeavors. I am a software engineer with two years of experience working as a backend engineer, now aiming to become a penetration tester. I began my offsec studies in early 2021, having attained the eJPT (eLearnSecurity's Junior Penetration Tester) by the end of the year, and now preparing for the OSCP! I created this page to share my findings and some walkthroughs (mainly in HackTheBox). I hope you can learn anything from this content."
permalink: /
image: assets/images/profile.png
nav_exclude: true
---

# Recent activity
<ul class="posts-box">
{% assign sorted = site.pages | sort:'date' | reverse %}
{% for page in sorted %}
{% if page.grand_parent == 'Capture the Flag' or page.parent == 'Hack The Box' or page.parent == 'blog' %}
    <li class="post_item">
      <div class="post">
        <div class="post_image"><img src="{{ page.image }}"></div>
        <div class="post_content">
          <h2 class="post_title">{{ page.title }}</h2>
          <p class="post_description">{{ page.description }}</p>
            <a href="{{ page.url }}">
                <button class="btn post_btn" type="submit">LINK</button>
            </a>
        </div>
      </div>
    </li> 
{% endif %}
{% endfor %}
</ul>