---
title: Using AzCopy/DataMovement libraries for SMB copy
description: >-
  I got an interesting one today. Copy a multi-terabyte file from on-prem to
  Azure. Problem is, the internet pipe is only 100mbps, while the…
date: ''
categories: []
keywords: []
slug: ''
---

I got an interesting one today. Copy a multi-terabyte file from on-prem to Azure. Problem is, the internet pipe is only 100mbps, while the pipe the VPN to Azure is using is 1Gbps (and basically unused, since this is early in their deployment).

Usually Azcopy is a slam dunk here — great for large files, resumability support, multithreaded, very feature-rich. Problem is that while I’m copying from on-prem to Azure, I can’t copy to blob because the internet pipe in this case is too small. We’d love to use the VPN pipe, but since there is no private addressability for storage, and we can’t modfiy the routes to include the public IP space for Azure, we have to get creative.

### DataMovement library

The DataMovement library is the [core data movement lib of Azcopy.](https://github.com/Azure/azure-storage-net-data-movement) This means the same things we get from azcopy — resumability, high-performance, etc — we can also use within DMLib.

Problem with DMLib is that it is assuming all your files are going to Azure Storage — which is something I’d love to do here but don’t have the option.

### Digging into DMLib

Looking into the data management library, we quickly find some interesting pieces — `TransferManager` for one, plus collections of `TransferReaders` and `TransferWriters`. In particular, `StreamedReader` and `StreamedWriter`. Since you can use azcopy for both downloading _and_ uploading files to Azure Storage, this makes sense — we’ll have to both read and write to the filesystem at different times.

In reality, all the bits are there for everything we need — the `TransferManager` includes methods for both upload and download, usually taking in a file/stream and cloudblob tuple for either upload or download.

There are no `UploadAsync` overloads that take two streams or files, however.

### Adding our own UploadAsync(string sourcePath, string destPath)