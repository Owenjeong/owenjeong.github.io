---
#title: "Splash Page"
layout: splash
permalink: /
# date: 2016-03-23T11:48:41-04:00
hidden: true
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/coffee.jpg
#   actions:
#     - label: "Download"
#       url: "http://google.com"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
excerpt: "Economy, Programming and Thought"
intro:
  - excerpt: '# Reflections and Research: A Deep Dive into My Studies and Thoughts'
feature_row:
  - image_path: assets/images/feat1.jpg
    image_caption: "Image courtesy of [Unsplash](https://unsplash.com/)"
    alt: "placeholder image 1"
    title: "Economy"
    excerpt: "Decoding the **Economy**: An Insightful Exploration of Facts and Personal Perspectives"
    url: "/categories/"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/feat2.2.jpg
    image_caption: "Image courtesy of [Unsplash](https://unsplash.com/)"
    alt: "placeholder image 2"
    title: "Programming"
    excerpt: "Deciphering Code: My Observations and Insights on **Programming**"
    url: "/categories/"
    btn_label: "Read More"
    btn_class: "btn--primary"
  - image_path: /assets/images/feat3.1.jpg
    image_caption: "Image courtesy of [Unsplash](https://unsplash.com/)"
    title: "Others"
    excerpt: "Musings and More: A Collection of My Thoughts and Experiences"
    url: "/categories/"
    btn_label: "Read More"
    btn_class: "btn--primary"

# feature_row2:
#   - image_path: /assets/images/feat2.jpg
#     alt: "placeholder image 2"
#     title: "Placeholder Image Left Aligned"
#     excerpt: 'This is some sample content that goes here with **Markdown** formatting. Left aligned with `type="left"`'
#     url: "#test-link"
#     btn_label: "Read More"
#     btn_class: "btn--primary"
# feature_row3:
#   - image_path: /assets/images/feat2.jpg
#     alt: "placeholder image 2"
#     title: "Placeholder Image Right Aligned"
#     excerpt: 'This is some sample content that goes here with **Markdown** formatting. Right aligned with `type="right"`'
#     url: "#test-link"
#     btn_label: "Read More"
#     btn_class: "btn--primary"
# feature_row4:
#   - image_path: /assets/images/feat2.jpg
#     alt: "placeholder image 2"
#     title: "Placeholder Image Center Aligned"
#     excerpt: 'This is some sample content that goes here with **Markdown** formatting. Centered with `type="center"`'
#     url: "#test-link"
#     btn_label: "Read More"
#     btn_class: "btn--primary"
---


{% include feature_row id="intro" type="center" %}

{% include feature_row %}


<!--
{% include feature_row id="author_profile" type="left" %} 
{% include feature_row id="feature_row3" type="right" %} -->