---
layout: post
title:  "Kubernetes"
date:   2021-04-16 11:00:00 +0200
categories: architecture hosting 
---

# Nomenclature
Deployments - Services
Pods - Instances
Services - Network


# Commands

kubectl get deployments (services)
kubectl get pods (pods = instanser)
kubectl get service (service = nätverk)

Logs:
kubectl get pods -o wide
kubectl logs [PodName] [ContainerName]

# Deployment
