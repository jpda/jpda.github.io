---
title: Sign in with Apple & Azure AD B2C
description: >-
  Last week at WWDC, Apple announced Sign in with Apple, a privacy-focused
  identity provider for your mobile apps & websites.
date: ''
categories: []
keywords: []
slug: ''
---

Last week at WWDC, Apple announced [Sign in with Apple](https://developer.apple.com/sign-in-with-apple/), a privacy-focused identity provider for your mobile apps & websites. 

Clearly, with anything new, we should throw caution to the wind and get cracking. Let’s go.

Overall, we’re going to 

*   register an app with apple’s idp
*   add apple as an idp in a b2c tenant

### Apple Developer Portal app configuration

First we need to register an app with Apple. I found this super un-intuitive but that may just be because I’ve never built or released an iOS app — regardless, it feels a bit wonky to a newcomer like me. [Aaron Parecki](https://medium.com/u/ae730c274a1) has a brilliant walkthrough here to get an app up and rolling