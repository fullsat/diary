---
title:  "Set up this site"
date:   2018-06-06 17:58:00 +0900
categories: tech
tags: jekyll
layout: post
---

## Purpose

I want to make a static website about my work.

## Environment

AmazonLinux2017.9

## Install

```
yum remove rubygem20 ruby20 ruby20-irb
yum install ruby23 ruby23-devel
yum groupinstall "Development Tools"
gem install jekyll
```

## GitHub Pages

1. repository 
2. Commit a something to master
3. settings tab
4. set branch

**You need to edit baseurl to "[repository name]" from "" in _config.yml.**

## Behavior of Jekyll

1. load _config.yml
2. build files
3. output to _site/

## Behavior on GitHub

If pushed repository is a Jekyll project, GitHub Pages behave as above process.
