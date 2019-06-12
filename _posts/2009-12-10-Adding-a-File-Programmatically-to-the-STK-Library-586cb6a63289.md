---
title: Adding a File Programmatically to the STK Library
description: ''
date: '2009-12-10T18:33:00.000Z'
categories: []
keywords: []
slug: /adding-a-file-programmatically-to-the-stk-library-586cb6a63289
---

![STK_Image_1](/img/0_7RZ1_RbMgVyDZTo8.png)

I’ve been working with the SharePoint Training Kit ([http://slk.codeplex.com](http://slk.codeplex.com)) Training Library recently, and one of the more important features was to create an uploader. The uploader was created in Silverlight (which I’ll get to later), but once we’ve got the file, what do we do then to make sure it gets added to the training library properly? Have a look below — most important is to set the content type.

These fields need to be set. The most critical of fields is the content type — which we set programmatically. **The file has to have its content type changed and** fileItem.Update() **called before setting any of these other properties.**

//add file to document library

folder.Files.Add(destFullPath, stream, true);

folder.Update();

web.Update();

//get file which we just uploaded and its associated SPListItem

SPFile file = folder.Files\[destFullPath\];

SPListItem fileItem = file.Item;

fileItem\[“ContentTypeId”\] = “0x010100EFCD84CEB66ABF4E9F9CA8F6C94312F3”;

fileItem.Update();

//set other properties and save

fileItem\[“Content”\] = “Interactive”;

fileItem\[“Sequence”\] = “0”;

**…**

fileItem.Update();

**Content Type**: 0x010100EFCD84CEB66ABF4E9F9CA8F6C94312F3

**Publish**: Make available to trainees

All of these fields are arbitrary and only for display — **they don’t affect processing or treatment of training items on the back end.**

**Name**: currently set to file name; does that make sense?

**Content**: three possible settings: Article, Interactive and Video — we can add\\remove as needed

**Sequence**: for multi-part items.

**Tooltip**: if separate tooltip text is desired.