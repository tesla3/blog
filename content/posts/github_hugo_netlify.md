---
title: "How to blog using github,hugo, netlify"
date: 2019-05-19T12:22:19-07:00
draft: false
---

I am late, very late, to the game of blogging. But, any way, I start to have something to say. Inspired by [this post] (https://blog.omerh.me/post/2019/05/17/first_post/) and [this post] (https://brainfood.xyz/post/20190518-host-your-own-blog-in-1-hour/), I decided to get it going. 

# Main idea
- github: to host and track revision of contents
- hugo: static content generator
 - written in go
 - high performance (read: fast)
 - CLI friendly
- netlify: content distribution network (CDN)

This setup gives us great flexibility, comparing to other managed such as wordpress or medium [ref]

# Set up
##  Hugo
### Install hugo and setup a new site
Since I am on Mac, brew is the easiest way to go
```bash
brew install hugo
hugo new site [path]
```


### Download a theme 
Go to [hugo themes] (https://themes.gohugo.io/) and pick one. I picked zen

```bash
cd [path]
git submodule add https://github.com/frjo/hugo-theme-zen.git themes/zen
```

### Bootstrap  config.toml
```bash
cp themes/m10c/exampleSite/config.toml .
```

Update copied `config.toml`, pay attention to `themeDir` and `theme`
```toml
baseURL = "https://example.com"
title = "Ramblings from Neo"
themesDir = "themes"
theme = "m10c"
paginate = 5

languageCode = "en-us"
enableRobotsTXT = true
googleAnalytics = ""
disqusShortname = ""

[params]
  author = "Thomas A. Anderson"
  description = "Thoughts on behavior and intelligence of human and computer"

```

### Test out
```bash
hugo server
```

Checkout your how your blog looks like at localhost:1313


## github
### Create a repo in github

You will get a `[git_repo_url]` e.g. `https://github.com/tesla3/blog.git`

### initiate local repo,  setup upstream, push local repo to remote 
```bash
cd [path]
git init
git add *
git commit -m "init commit"
#now master branch is created

#setup upstream
git remote add origin [git_repo_url]
#e.g.
#git remote add origin https://github.com/tesla3/blog.git

git push
```


## Connect netlify to your github repo

### Flow [step-by-step](https://medium.com/the-codelog/how-to-deploy-a-website-to-netlify-35274f478144)

It is pretty straightforward. Only things specific to hugo: at step of "Basic build setting", put **hugo** in **Build command** and **public/** in **Publish directory**


* Note: it seems netlify figures out by itself the repo is a hugo, so both build command and publish directory is auto filled if you have pushed hugo repo to github before linking netlify to your github 

### Now whenever you push to your git repo master branch, netlify will rebuild and publish the site for you.

Here is how you can check your **deploy** status from netlify





