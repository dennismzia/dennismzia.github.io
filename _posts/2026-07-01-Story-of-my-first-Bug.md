---
title: STory of My first Paid Bug
description: This blog details my journey of uncovering a series of vulnerabilties in a management system
author: Dennis_Mwanzia
date: 2026-07-01 10:18:18 +0000
categories: [Cybersecurity]
tags: [security, IDOR, hacking]
pin: false
math: false
mermaid: false
# image:
#   path: https://assets.hemantapkh.com/blog/seedr-xss/thumbnail.webp
#   alt: Image generated with DALL-E 3
---

# Story of My First Bug

In 2020, during the COVID-19 pandemic, significant global changes occurred. On a positive note, this period saw widespread adoption of online work and shopping, which subsequently brought increased awareness to online risks and the loose term "hacking."

That February, after completing most of the labs on the PortSwigger platform, I decided to try my luck with "bug bounty hunting." This story covers how I began that journey.

The program I chose was a learning management platform, which we will refer to as examplelms.com. As a beginner, I was initially overwhelmed by the analytics and tracking data. Instead of being discouraged, I decided to use the application to teach me. I would encounter headers like X-FRAME-OPTIONS, research their purpose, and learn why they existed—a practice I have come to call "just-in-time learning."

After familiarizing myself with the application, I noticed a feature where instructors could create both free and premium courses. While learners could naturally access free courses, they would encounter a paywall after "previewing" a paid course. I discovered a simple IDOR vulnerability: by replacing the free course ID with the ID of a paid course in the endpoint, I could access the premium content for free.

While not as technical as my current research, it was a great start into the world of security. I realize this was neither complex nor new, but it serves to reinforce the idea that some exploits—even some major hacks—are not inherently complicated. After a long career in this field, I have learned that even the most complex hacks usually just chain together simple primitives to achieve massive impact.

I appreciate you reading this. If you would like to connect with me, you can find me on my socials. Like those before me, if I have gone far, it is because I have stood on the shoulders of giants. I started this blog to share my ideas and connect with other hackers in the industry.
