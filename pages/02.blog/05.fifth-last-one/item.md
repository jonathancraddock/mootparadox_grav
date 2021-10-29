---
title: 'Fifth (last one)'
visible: false
taxonomy:
    category:
        - coding
    tag:
        - code
        - test
        - syntax
routes:
    aliases:
        - /coding/test
---

## Five

And that should be the last before starting pagination?

===

Code snippet, any syntax highlighting?

```twig
<section class="taxonomyCats">
  {% include 'partials/taxonomylist.html.twig' with {base_url: '/blog', taxonomy: 'category'} %}
</section>
```