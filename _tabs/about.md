---
# the default layout is 'page'
# tabs are not searchable, only posts
icon: fas fa-info-circle
order: 4
---

This site is a personal log of things related Amateur Radio, with a focus on the non-transmitting options accessible unlicenced SWL/Short Wave Listeners.

- [x] Member of [the IRTS](https://irts.ie)
- [x] Member of [the National Shortwave Listeners Club](https://swl.ie)

The theme used provides a search feature, which works on posts, the Archive provides a timeline of posts, and tags/categories work just as you'd expect.

## Website build

This is site is built using [Github Pages](https://pages.github.com/) for code managment, hosting, and the build/deploy pipeline (actions). 

Using those, it is generated from the [Jekyll static site generator](https://jekyllrb.com).

It builds directly upon the [the Chirpy theme](https://github.com/cotes2020/jekyll-theme-chirpy?tab=readme-ov-file#chirpy-jekyll-theme) for Jekyll, which thankfully handles all of the behind the scenes integration with Jekyll, Github actions.

The whole install, setup and deploy was done in minutes using [their getting started notes](https://chirpy.cotes.page/posts/getting-started/#option-1-using-the-starter-recommended), which showed how to use [their template repo](https://github.com/cotes2020/chirpy-starter) on github as a starting point in just a few clicks.

> All of the github pages related integration including Jekyll build (generating static output) and pages-deploy was handled behind the scenes just by following those instructions.
{: .prompt-info }

What takes a bit longer is figuring out how the theme works, how Jekyll works, customising your posts and configuration & behaviour. 

It is possible to speed up that try, test and learn processing by installing a local build/deploy stack [which i wrote about here]({% post_url 2024-08-26-local-jekyll-for-webpage-testing %})