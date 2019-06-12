---
title: Adding Code-Behind to User Controls in SharePoint
description: ''
date: '2010-04-27T19:49:00.000Z'
categories: []
keywords: []
slug: /@jpda/adding-code-behind-to-user-controls-in-sharepoint-3f3b030e4de7
---

This question came from SharePoint Overflow ([http://sharepointoverflow.com](http://sharepointoverflow.com)). The question was — using the control templates that dictate layout of assorted forms, how can we add custom code to them?

The answer is fairly straightforward. Here we go:

First, get your ascx ready. This is a two step process:

Remove the ‘CodeBehind=’ declaration in the top of your ascx.

![code1](https://cdn-images-1.medium.com/max/800/0*WXqZS4ujjSJk6s79.png)

Next, make sure your classes and namespaces are named properly.

![code2](https://cdn-images-1.medium.com/max/800/0*qclIse7xeXIRkUlo.png)

Build the project.

Drop your ascx into the template\\controltemplates folder (or wherever you want them, you may need to adjust your CAS).

Drop your built, signed assembly somewhere (probably the GAC, or wherever else your CAS has proper trust).

In this example, we’re changing the DefaultTemplates.ascx file — probably a good idea to back this file up first.

![code3](https://cdn-images-1.medium.com/max/800/0*fKSfL-oooC-It1Nz.png)

Go to the template you’re modifying, and add your controls appropriately, with a Register tag in the top and the standard ASCX markup (i.e, <TagPrefix:MyControl runat=”server” />).

Hit your page and your control should appear without issue.

![code4](https://cdn-images-1.medium.com/max/800/0*TxK7DtWefY_IAqpW.png)
![code5](https://cdn-images-1.medium.com/max/800/0*vbAUyqNYgsi-9kfY.png)