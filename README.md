#Introduction

This blog can be seen on 
Welcome to the sources of my jekyll based [hawkbit blog](ddewaele.github.io/hawkbit-blog) covering [Hawkbit](https://projects.eclipse.org/proposals/hawkbit).
I'll be posting some docs / code samples here related to [Hawkbit](https://projects.eclipse.org/proposals/hawkbit).

References

- [Hawkbit Project Page ](https://projects.eclipse.org/proposals/hawkbit).
- [HawkBit Git Repo](https://github.com/eclipse/hawkbit).

## Theme used

The Jekyll theme used on this blog is bsed on the [hpstr-jekyll-theme](https://github.com/mmistakes/hpstr-jekyll-theme/).


### Customizations

I've customized the theme a little bit :

- in `_sass/page.scss` I've increased the max-width from 800px to 1200px
- updated the license
- updated the config (change title, discuss, url , name, )
- removed adds from the footer
- updated the `about/index.md` page
- updated the `index.html` page to show `post.excerpt` instead of `post.content`
- Using Read More links in the post (for post excerpts)

If you want to clone this blog and run it locally (for whatever reason), simply run

```
bundle exec jekyll s  -c _config.yml,_localhost_config.yml --drafts
```