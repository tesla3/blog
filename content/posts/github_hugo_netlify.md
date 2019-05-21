---
title: "Blog using github,hugo, netlify"
date: 2019-05-19T12:22:19-07:00
draft: false
---
update (Tue May 21 12:15:06 PDT 2019) : [kubeflow website](https://github.com/kubeflow/website) uses same set up.

I am late, very late, to the game of blogging. But, any way, I start to have something to say. Inspired by [this post] (https://blog.omerh.me/post/2019/05/17/first_post/) and [this post] (https://brainfood.xyz/post/20190518-host-your-own-blog-in-1-hour/), I decided to get it going. 

# Main idea
- **github**: to host and track revision of contents
- **hugo**: static content generator
 - written in go
 - high performance (read: fast)
 - CLI friendly
- **netlify**: content distribution network (CDN)

This setup gives us great flexibility, comparing to other managed blog platform such as wordpress or medium. 

# Set up (local)
##  Hugo
### Install hugo and setup a new site
Since I am on Mac, brew is the easiest way to go
```bash
brew install hugo
hugo new site [path]
```

### Download a theme 
Go to [hugo themes] (https://themes.gohugo.io/) and pick one. After some trail and error, I picked [Coder](https://themes.gohugo.io/hugo-coder/) for its cleanness and simplicity.

```bash
cd [path]
git submodule add https://github.com/luizdepra/hugo-coder.git themes/hugo-coder
```

### Bootstrap 
copy everything from `themes/huge-coder/exampleSite/` to `[path]`. Customize `config.toml`

* baseurl has to set up properly before checking into github and build to deploy into netlify if `canonifyurls` is set to `true` 

Update copied `config.toml`, pay attention to `themeDir` and `theme`


### Test out
```bash
hugo server
```

Head to `localhost:1313` to check out how the blog 


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

It is pretty straightforward. Only things specific to hugo: at step of "Basic build setting", put **hugo** in **Build command** and **public/** in **Publish directory**


* Note: it seems netlify figures out by itself the repo is a hugo, so both build command and publish directory is auto filled if you have pushed hugo repo to github before linking netlify to your github 

Now whenever you push to your git repo master branch, netlify will rebuild and publish the site for you.

It is also straightforward to check your **deploy** status from [netlify](https://app.netlify.com/sites/)



