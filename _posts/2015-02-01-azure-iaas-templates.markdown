---
layout: post
title: "Azure Virtual Machine Templates"
date: 2015-02-01 14:42:41 +0000
comments: true
categories: [Azure, virtual machines, cloud]
---

Azure Virtual Machines (Or if you live in buzzword land, Infrastructure-as-a-service) is a great way to quickly spin up new environments. But configuring these environments can be a right pain. 

A quick and easy way to provision Virtual Machines is to use templates via VM Depot or Microsoft.

<!-- more --> 

The [VM Depot](http://vmdepot.msopentech.com) is a community-driven collection of templates for Virtual Machines running in Azure. They can be created automatically or using the Management Portal from the VM Depot images, and you can contribute new images as well.

It's worth checking out, head on over and use the comprehensive image search to find what you're after.

I've been wanting to checkout Visual Studio 2015 and ASP.NET vNEXT for a while, instead of downloading an installer, waiting hours and building a new Windows Server 2012 VM and then deleting it when I'm done, I've been using Azure to deploy VS2015 images.

In the "Create an Image" section of the Virtual Machine wizard, I select "Visual Studio Ultimate 2015 Preview".

![My helpful screenshot]({{ site.url }}/images/azure-vm-choose-an-image.png)

This is where it gets interesting. I've gone for a D series, they run on a SSD and are blazing fast. Remember, you only pay for compute time (i.e. when the VM is running). At the time of writing, a D2 (2 cores, 7Gb memory) cost £0.20 an hour. Basically I am renting hardware.

![My helpful screenshot]({{ site.url }}/images/azure-vm-configuration.png)

If I use the VM 5 hours a week, that's £1 a week or £52 a year. What's more cost effective? Buying hardware to run it, or just remote desktop to my instance? It's worth thinking about. For more details on Microsoft's pricing on Azure VMs go [here](http://azure.microsoft.com/en-gb/pricing/details/virtual-machines/).

All that's remaining is to remote desktop in and start to use ASP.NET vNEXT on Visual Studio 2015, hurrah!

![My helpful screenshot]({{ site.url }}/images/azure-vm-configuration.png)