---
title: 'Redirect a Subset of Pages Using .htaccess'
taxonomy:
    category:
        - coding
    tag:
        - htaccess
        - redirect
        - rewrite
        - apache
    coding-cat:
        - web
---

Wrestling with .htaccess syntax is a pleasure that I engage in every six months or so! I grasp the basic principles, but have never had cause to become fully fluent. These two rules are the culmination of a requirment to redirect pages from an old site that used different URI structures; with and without query strings.

===

There were two 301 permanent redirects that I wanted to implement:

* https://example.com/blog/firstpost -> https://newsite.com/information/first-post-on-this-blog
* https://example.com/index.php?sec=blog&post=secpost -> https://newsite.com/information/second-post-here

Sounds simple, but I was unhappy with my initial attempt because it was retaining the query string after the rewrite. Even worse, the legacy code on the old site was already rewriting URIs such as `/blog/firstpost` into `/index.php?sec=blog&post=firstpost` and I didn't want that rearing its ugly head.

After an amount of head-scratching, the following two instructions achieve what I wanted. (I'm not an Apache expert, and there may be a better way.)

**Example 1:**

```bash
# leagcy site 301 redirects
RewriteRule ^blog/firstpost/?$ https://newsite.com/information/first-post-on-this-blog [NC,L,R=301]
```

I included this rule near the top of the .htaccess (obviously after the RewriteEngine On, etc) and before the legacy rules that were manipulating various query strings.

**Example 2:**

```bash
# Old Links
RewriteCond %{QUERY_STRING} post=secpost
RewriteRule ^index\.php$ https://newsite.com/information/second-post-here/? [NC,L,R=301]
```

Apache 2.4 introduced the `QSD` flag, Query String Discard. The final ? in the redirect above predates that, but it works and I didn't feel concerned by it.

It turned out to be fairly straightforward in the end. Lots of examples that I found are using regex expressions to redirect whole sites, which is great, but in this case a limited subset of pages required the redirect. Also the destination URI was very different. It's possible to implement a route alias on the new page, but that's displayed in the address bar, and I didn't like it.

Good luck! ;-)