+++
title = "Moving the blog to Hugo"
date = "2016-11-26T16:03:03+01:00"
slug = "moving-the-blog-to-hugo"
categories = ["Blog", "Technology"]
tags = ["Blog", "Hugo", "Git", "GitHub", "CloudFlare", "HTTPS", "HSTS", "Octopress", "Let's Encrypt"]
+++

Quite a chunk of time has passed since my last update to this blog. While primary reason is my super busy life (on both proffessional and personal fronts) there was also another one I have finally addressed, along with a few extras. The reason was that over the years I simply grew tired of WordPress software. The point to migrate away from it occupied my TODO list for too long. I am not going to bash WP here, I used it for many years, and for that I am grateful to its authors. However, for me at least the time has come to move towards something else.

## Back to simplicity

I remember that when I was in high school my friend was running entire e-commerce site using static HTML generator written in Perl, combined with some CGI scripts to chandle checkout process. Going even further back, I remember the times when I actually used notepad to edit HTML 3 files, and upload that with FTP over a 28k modem connection to a hosting provider. Things were simple back then, and I truly miss that simplicity.

Having to run entire LAMP stack on my own just to run simple website is standing contrary to the rule of keeping things simple. That's exactly what I was doing for the past few years, and that's what I wanted to change. This is the primary reason I decided to look at static website generators as a new blogging platform. All these CMS systems have been created for people who don't know their way around computers. Hackers know better :)

There is quite a few modern generators to choose from. My initial favorite was [Octopress](http://octopress.org/), but eventually I settled on [Hugo](https://gohugo.io/) but with Octopress-inspited theme. Hugo is simple, easy to customize, trivial to install, comes with built in webserver that watches file changes, and reloads the pages in your browser just when you save source file via JavaScript magic. It's also very, very fast - regenerating this website in its entirety takes 200-300 ms on my MacBook Pro. 

Hugo, similar to other generators, uses Markdown as source file format. I guess the format has been popularized by GitHub (and likely other hosted Git solutions) with automatic rendering of README.md in the repositories. Regardless of the reason, it is well known and liked by hacker types. It has some limitations, but also lets you focus on the content and not its form. With Hugo you can easily work around Markdown limitations by using your own macros that expand to HTML during rendering. So, while 99% of the time sticking to plain Markdown syntax is enough, you are not left alone in the corner cases.

## GitHub Pages

GitHub has launched [GitHub Pages](https://pages.github.com/) quite a while ago. It's totally free service that let's you serve static web content to the world. There are traffic limitations but not at the level that matters for a personal blog. Should you somehow run into these limitations, there are plenty of other options, Amazon S3 among them. For now, sticking to GitHub gives me very nice publishing flow. In your website directory:

{{< highlight bash >}}

MaciekMBP:hugo maciek$ hugo
Started building sites ...
Built site for language en:
0 draft content
0 future content
0 expired content
35 pages created
0 non-page files copied
6 paginator pages created
45 tags created
17 categories created
total in 227 ms
MaciekMBP:hugo maciek$ cd public
MaciekMBP:public maciek$ git add -A
MaciekMBP:public maciek$ git commit -m "New version info"
MaciekMBP:public maciek$ git push

{{< /highlight>}}

That's all, new site version is live!!! Of course you can (and should :) ) use version control tools for your source files, too. For ambitious types: implement building and publishing of your website via continuous integration server :)

## CloudFlare

Serving static website from GitHub is very quick, and supports HTTPS within github.io domain. What if you want HTTPS on your own domain? Here is where [CloudFlare](https://www.cloudflare.com/) steps in. Within their **free** plan you can get not only free HTTPS, but with great options like: redirecting HTTP straffic to HTTPS, turning on HSTS (HTTP Strict Stransport Security) for the site, or even forcing TLS 1.3 use (beta feature). While going crazy with security on personal blog that's available on GitHub anyway may seem like an overkill, I strongly believe that in the era of [Let's Encrypt](https://letsencrypt.org/), non-encrypted HTTP traffic should just die. Using CloudFlare has an additional advantage of speed: it's not just about encryption, but also a CDN sitting in front of GitHub. To make it even sweeter, CloudFlare fully supports HTTP/2 and SPDY protocols.

And one more thing: thanks to CloudFlare [CNAME Flattening](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/) you can have CNAME at top level of your domain **without breaking the RFC**. As a result, the address for this blog from now on will be just https://maciejmroz.com, served to you at zero cost in "cloud native" way :)