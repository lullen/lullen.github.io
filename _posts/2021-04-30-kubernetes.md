---
layout: post
title:  "Kubernetes - Local development"
date:   2021-03-30 11:00:00 +0200
categories: architecture hosting 
---

This post is not supposed to show best practices or anything, it will show what I have learnt about k8s in local development and I will be updating this post as I learn more. The tools I have decided to use as a base are [k3s](https://k3s.io), [skaffold](https://skaffold.dev) and [dapr](https://dapr.io).


By this time everyone knows what k8s is so I won't bore you explaining what it is. I have been using Azure ServiceFabric since 2016 and have always been a big fan (and still am) but since .NET core was released the cost of Windows has always seem high and have been wanting to move to Linux. As Azure ServiceFabric is basically Windows only, I have never been able to do the migration.

Kubernetes is now more or less the standard and the I believe that the sooner we can stop talking about orchestrators the better. An orchestrator should be something that's something behind the scenes so we can focus on things that matter, the business value. One reason of worry is that none of the big giants are using k8s as the orchestrator for their cloud platform but as the usage is widespread I think that worry isn't that as big but in general it's always a good thing to see what the big giants are using instead of what they are selling.

Kubernetes is also doesn't give you a programming model but instead it lets you do whatever you want. I think it's important to have a programming model that you start with and here is where [dapr](https://dapr.io) comes into play. Although it is a young project I have been following it closely since it was announced and it already gives you a lot that you would need to write yourself like service discovery, retry policies, observability, security and more. It will be very interesting to follow the project to see what their plans are.

# Nomenclature
- Deployments - Services
- Pods - Instances
- Services - Network

# Commands
- kubectl get deployments
- kubectl get pods
- kubectl get service

Logs:
- kubectl get pods -o wide
- kubectl logs [PodName] [ContainerName]

# Getting started

{% highlight bash %}
curl -sfL https://get.k3s.io | sh -

# To be able to run kubectl without sudo
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644

# Check for Ready node, takes maybe 30 seconds

k3s kubectl get node

{% endhighlight %}

# Skaffold
Skaffold is a tool that originated from Google. They describe the tool as "Skaffold handles the workflow for building, pushing and deploying your application, allowing you to focus on what matters most: writing code."

To get started you run `skaffold init` at the root of your project directory and follow their wizard.

# Development
To get everything up and running, you only need to run one command:
{% highlight bash %}
skaffold dev
{% endhighlight %}
