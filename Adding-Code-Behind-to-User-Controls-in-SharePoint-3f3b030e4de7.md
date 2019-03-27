---
title: "Adding Code-Behind to User Controls in SharePoint"
date: 2010-04-27T19:49:00.000Z
author: "John Patrick Dandison"

---

This question came from SharePoint Overflow ([http://sharepointoverflow.com](http://sharepointoverflow.com)). The question was — using the control templates that dictate layout of assorted forms, how can we add custom code to them?

The answer is fairly straightforward. Here we go:

First, get your ascx ready. This is a two step process:

Remove the ‘CodeBehind=’ declaration in the top of your ascx.




![code1](http://jpd.ms/wp-content/uploads/migrated/code1_thumb.png)



Next, make sure your classes and namespaces are named properly.




![code2](http://jpd.ms/wp-content/uploads/migrated/code2_thumb.png)



Build the project.

Drop your ascx into the template\controltemplates folder (or wherever you want them, you may need to adjust your CAS).

Drop your built, signed assembly somewhere (probably the GAC, or wherever else your CAS has proper trust).

In this example, we’re changing the DefaultTemplates.ascx file — probably a good idea to back this file up first.




![code3](http://jpd.ms/wp-content/uploads/migrated/code3_thumb.png)



Go to the template you’re modifying, and add your controls appropriately, with a Register tag in the top and the standard ASCX markup (i.e, &lt;TagPrefix:MyControl runat=”server” /&gt;).

Hit your page and your control should appear without issue.




![code4](http://jpd.ms/wp-content/uploads/migrated/code4_thumb.png)





![code5](http://jpd.ms/wp-content/uploads/migrated/code5_thumb.png)
