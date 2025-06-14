---
title: Aspnet core data protection
description: >-
  Let's supposed that you need to persist trusted information for later retrieval, but you don't trust the persistence mechanism. In web terms, this might be written as I need to round-trip trusted state via an untrusted client.
author: sergioacortes
date: 2025-06-13 00:00:00 +0800
categories: [Blogging, Aspnet, Data protection]
tags: [Aspnet]
pin: true
media_subpath: '/posts/20250613'
---

# Use case: Running the application behind a reverse proxy.

Let's imagine an application running on kubernetes with multiple replicas running simmultaniusly, as you might know, in kubernetes there is an ingress that redirect the traffic to the different replicas. You also might know, that if your kubernetes instance is running on a cloud provider ([Microsoft Azure](https://portal.azure.com/), [Amazon Web Services](https://aws.amazon.com/es/), [Google Cloud Platform](http://cloud.google.com/), etc), there might be a load balancer. In resume, the application is running behind a reverse proxy.

In our use case, the application that is running behing a reverse proxy, start an authentication flow with the identity server (OpenIdConnect) and needs to save the state to complete the authentication flow once the identity server sends the response. The problem faced in this use case comes because the different replicas running in kubernetes are not sharing the state and there is not guarantee of wich replica will attend the identity server response, here is when [Aspnet core data protection](https://learn.microsoft.com/en-us/aspnet/core/security/data-protection/introduction?view=aspnetcore-9.0) comes into play.

 