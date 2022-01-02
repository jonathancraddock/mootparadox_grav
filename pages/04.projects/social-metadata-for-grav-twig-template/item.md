---
title: 'Social Metadata for Grav Twig Template'
date: '02-01-2022 18:45'
media_order: '00metadata.jpg,404-image.jpg'
taxonomy:
    category:
        - projects
    tag:
        - grav
        - twig
        - 'social media'
        - metadata
        - html
        - tags
    projects-cat:
        - web
---

As much as I dislike social media, if in the unlikely event somone ever wanted to *share* one of these posts, I'd like it to have an image and a description. Including some OpenGraph and Twitter metadata sounds like it ought to be sufficient.

===

I took the pragmatic approach of seeking out an article that looked correct after being shared on Mastodon (the only platform I actively use) and then examining the source to see what else they had included. As it happens, it was a news post from [Nextcloud](https://nextcloud.com/blog/), but I think the syntax is pretty standard.

With that as a starting point, it requires a few minor adjustments to turn it into a Twig template.

```twig
{% block metasocial %}

<!-- Open Graph data -->
  <meta property="og:title" content="{{ page.title }}" />
  <meta property="og:type" content="article" />
  <meta property="og:url" content="{{ page.url(true, true) }}" />
  <meta property="og:site_name" content="example.com" />
  <meta property="og:description" content="{{ page.summary|striptags|trim|e }}" />

<!-- Twitter Card data -->
  <meta name="twitter:title" content="{{ page.title }}" />
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:url" content="{{ page.url(true, true) }}" />
  <meta name="twitter:site" content="example.com" />
  <meta name="twitter:description" content="{{ page.summary|striptags|trim|e }}" />

<!-- Image data -->
{% set image = page.media.images|first %}
{% set imgdefault = "https://example.com/user/images/img-default.jpg" %}

{% if image %}
  <meta property="og:image" content="https://example.com{{ image.url|raw }}" />
  <meta name="twitter:image" content="https://example.com{{ image.url|raw }}" />
  <meta name="twitter:image:src" content="https://example.com{{ image.url|raw }}" />
  <meta itemprop="image" content="https://example.com{{ image.url|raw }}" />
{% else %}
  <meta property="og:image" content="{{imgdefault}}" />
  <meta name="twitter:image" content="{{imgdefault}}" />
  <meta name="twitter:image:src" content="{{imgdefault}}" />
  <meta itemprop="image" content="{{imgdefault}}" />
{% endif %}

{% endblock %}
```

In summary, the required fields are:

* Title -> `{{ page.title }}`  
* Canonical URL -> `{{ page.url(true, true) }}`  
* Description -> `{{ page.summary|striptags|trim|e }}`  

Images present a slightly special case, in that a post might not include an image. Here I'm using an equivalent sized 'default' image; just the website address on a soft background.

```twig
{% set image = page.media.images|first %}
{% set imgdefault = "https://example.com/user/images/img-default.jpg" %}
```

And then determine which image to use:

* If there's an image -> `https://example.com{{ image.url|raw }}`
* Without an image -> `{{imgdefault}}`

![404-image](404-image.jpg "404-image")

For testing, this [OpenGraph Preview](https://www.opengraph.xyz) page proved useful, and lets you view the output as seen by multiple sites, from Facebook to Linkedin.