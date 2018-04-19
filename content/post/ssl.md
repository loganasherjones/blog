---
title: "Certificates, TLS, and HTTPS for Pragmatic Developers"
date: 2018-04-14
categories:
  - python
  - security
tags:
  - python
  - development
  - security
  - certificates
  - ssl
  - tls
keywords:
  - python
  - security
  - tls
  - ssl
  - beginner
  - development
  - basics
thumbnailImagePosition: right
thumbnailImage: ./images/second.jpg
coverImage: ../../../images/second.jpg
coverCaption: Photo Credit https://www.katiehowellphotography.com/
showMeta: false
draft: true
---

This is the absolute minimum every developer should know about certificates, certificate authorities, HTTPS and TLS but are too afraid to ask.

<!--more-->

<!-- toc -->

# Introduction

If you're writing a web application, or if you're even interacting with a web application and do not understand certificates, I would like to encourage you to take a moment and read this article. I'm going to try to lay out the basics of certificates, their purpose and how they work without going into too many math details or cypher deep-dives. Instead, I'm going to focus on what errors you're likely to see, why they are happening and how you can fix them. Consider this a getting started guide for understanding how to work with certificates.

If you're already a security expert, and are expecting a talk about quantum key distribution, go away. This post isn't for you. If you keep getting errors like `[SSL: CERTIFICATE_VERIFY_FAILED] certificate_verify failed (_ssl.c:777)` and then run to StackOverflow to figure out what's going on, because you just want to download some package or hit some API and this error keeps getting in your way, then read on. This post is intended for you.

I'm going to be diving into a quick demonstration app using python. I'm going to show how certificates work from end-to-end, generate some certificates, even our own certificate authority and even include TLS client certificate authentication. Let's get started!

# TLS vs SSL vs HTTPS

Let's start by examining the difference between HTTPS, SSL, and TLS. Generally speaking, you may see people use these terms interchangeably, but I want to clear things up. They are not the same technologies and are meaningfully different. So let's explore them one-by-one.

SSL (or Secure Sockets Layer) is a cryptographic protocol that 

HTTPS (or it easier to pronounce and way-cooler sounding name Hypertext Transfer Protocol Secure)
