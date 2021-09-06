---
title: "Adding_Google_Analytics_and_AdSense_to_a_Hugo_website"
date: 2020-11-08T20:53:09+08:00
tags: ["hugo"]
keywords: 
- hugo
- blog
- 博客
- 建站
- 评论
description: "Hugo 添加Google 分析和广告"
categories: ["blog", "hugo"]
draft: false
---

# Adding Google Analytics and AdSense to a Hugo website

Hugo is a great static website generator, I use it for this blog and am absolutely loving it! However, the first issue that I came across was that the theme that I’m using LoveIt does not come with any configuration options for using Google Analytics or including Google AdSense ads. I also wanted to include some custom headers such as a favicon and some description lines.

It took a little reading and tinkering to get this working so I thought I would document how I got this working and hopefully save someone else some time.

The first thing to understand is that Hugo builds the website pages out of blocks of code. If you are using a straightforward theme like I am then the file at:

```
/hugo_root/themes/theme/layouts/_default/single.html
```

Is the file that builds the pages. This does not contain most of the HTML that will end up in the final page, rather, it contains calls to other files that contain the needed HTML. Notably, the `header.html` and `footer.html` files.

If we want to add some custom HTML into the page header then we will need to add it to the `header.html` or include a call to another file that does contain the HTML. We will do that latter as it will be much easier to keep everything neat if we put the Google Analytics code in its own file.

First, we need to create the files that will contain the code we want to include in the site’s pages.

We could simply create our new files in the theme directory along with all the theme’s files. However, this will create more work if you want to change the theme later on as you will need to move them into the new theme’s directory.

Instead, we will create a new directory under `hugo_root/layouts` which will be accessible by any theme. As all the following HTML files will contain Google related code we will put them in their own directory, `google`.

From the root directory of your Hugo site (shown as `hugo_root` in the paths) run the following command:

```
mkdir -p hugo_root/layouts/partials/google
```

Now you need to get your Google Analytics tracking code. You get this from:

```
Google Analytics -> Admin -> Property > Tracking Info -> Tracking Code
```

The tracking code looks like the following:

```
<!-- Global Site Tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-100000000-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments)};
  gtag('js', new Date());

  gtag('config', 'UA-100000000-1');
</script>
```

Copy and paste your code into a file ending in `.html` under your `hugo_root/layouts/partials/google` directory. The following command will do this with nano:

```
nano hugo_root/layouts/partials/google/analytics.html
```

Next, we are going to edit the `header.html` file so that the contents of `analytics.html` is included in the page headers. Hugo uses the following structure for including the contents of a file under the `hugo_root/layouts/partials/` directory:

```
{{ partial "google/analytics" . }}
```

The “google/analytics” means look in the `google` directory under the `partials` directory for a file named `analytics.html`. The `.html` file extension is omitted.

All we need to do is to add this in the correct location of the `header.html` file to get out Analytics code added to our pages.

The header file can be found at:

```
hugo_root/themes/theme/partials/header.html
```

The first few lines of the `header.html` file in the Minimal theme looks like:

```
<!DOCTYPE html>
<html lang="{{ .Site.LanguageCode }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{{ .Title }}</title>
        {{ partial "css" . }} {{ partial "js" . }} {{ .Hugo.Generator }}
        {{ if .RSSLink }}
        <link href="{{ .RSSLink }}" rel="alternate" type="application/rss+xml" title="{{ .Site.Title }}" />
```

You need edit the file so that it includes the Analytics code by inserting `{{ partial "google/analytics" . }}` into the headers, e.g.:

```
<!DOCTYPE html>
<html lang="{{ .Site.LanguageCode }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        {{ partial "google/analytics" . }}
        <title>{{ .Title }}</title>
        {{ partial "css" . }} {{ partial "js" . }} {{ .Hugo.Generator }}
        {{ if .RSSLink }}
        <link href="{{ .RSSLink }}" rel="alternate" type="application/rss+xml" title="{{ .Site.Title }}" />
```

Finally, re-generate your site and all your pages will have your Analytics code in the headers.

If you want to put AdSense ads into your pages it is the same process. Copy your AdSense code into a new `.html` file under `hugo_root/layouts/partials/google`.

Then edit the file that creates the pages, `hugo_root/themes/theme/layouts/_default/single.html`, to include your AdSense into the pages using the same `{{ partials "google/file" . }}` code.