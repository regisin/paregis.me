---
title: "Static Website Introduction"
date: 2021-10-12T00:00:01-05:00
draft: false
description: "Introducing a new tutorial series about implementing a static website from scratch with Python, and hosting it on GitHub."
categories:
- Tutorial
---

## Table of content

- [Table of content](#table-of-content)
- [Introduction](#introduction)
- [What to expect](#what-to-expect)
- [What is a static website?](#what-is-a-static-website)
- [So why generate static pages for a website?](#so-why-generate-static-pages-for-a-website)
- [Objective](#objective)
- [Summary](#summary)

## Introduction

I decided to write down a series of short(ish) blog posts show how to build a personal website (and host it!) using some common and simple tools. This series is mainly targeted for beginner programmers, but there will be some cool stuff that even advanced ones could benefit from. This will be a long series of posts, feel free to skip back and forth as you wish.

In this first post I will introduce some basic concepts and layout the motivation.

## What to expect

In this series I will show how to build a static website using `python` and `html`. It will be very useful if you already have some basic knowledge of both.

I will show how I created my personal website (an earlier version of it) and hosted it for free on [GitHub](httpos://github.com). I will also show to automate the development process so it's easier to update it with new projects and content. In a nutshell you will learn how to create a static website generator, and as a bonus it will have a continuous deployment workflow.

## What is a static website?

A static website is one that does not generate it's pages dynamically. For example, you know when you post something on social media and a few seconds later your friends can see it in their own feed the thing you just posted? That is a dynamic website. It requires some powerful servers to run those. You need a database, a webserver.. and probably a ton of other types of tools.

A static website, on the other hand, is a collection of files that a browser can fetch from a server. It does not change dynamically, if you want it to be updated, you will have to change those files yourself and replace them on the server, manually.

In this series I will make a static website with SOME level of automation. The pages will be generated offline before being served and not in real time like the social media example.

## So why generate static pages for a website?

It may seem counter-intuitive to even consider creating a static website after all these years with so many advanced tools and platforms out there. But I make the point that not all websites need to have a powerful backend and even a proper database. There are many use cases where dynamic content is not required at all, and having static pages gets the message across and saves valuable time that could be used in more meaningful tasks.

Personal websites are the perfect example. It's a personal website, it's not a full fledged web application. You might have a personal blog on your website, and that's totally cool. But let me ask you, how often do you update a blog? How often do you finish a new project to add to your personal portfolio? For most people, not that often.

## Objective

With this series we will learn how to use `python` and `html` to build a personal website. There are other static website generators out there, such as [Jekyll](https://jekyllrb.com/), but the purpose of this series is to make your life even easier, without having to learn yet another tool (Jekyll) to create the website. For this workshop, all you need to know is some basic `html` and `python`. I will use [GitHub](httpos://github.com) to host the files online, and use ~~Travis-CI~~ ~~CircleCI~~ [GitHub Actions](https://github.com/features/actions) to simplify the publishing process when you update the website contents.

## Summary

This series targets folks that already have some knowledge of how the web works. You know a little bit of `python`, how to create a script, how to load and read text files from a script. I will cover:

- Setting up the development environment
- Creating `html` templates using `Jinja2`
- Create a data store (not data base!)
- Combining templates and data from the store to create pseudo dynamic pages
- Publish the website to GitHub using Actions