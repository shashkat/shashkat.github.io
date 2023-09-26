---
title: "🕸️ Predicting stability of Gene Networks"
layout: post
date: 2023-01-31 22:10
tag: 
- Neural Networks
- Systems Biology
image: assets/images/gene_network.png
# headerImage: true
projects: true
hidden: true # don't count this post in blog pagination
description: "My work in predicting the stability of gene networks."
category: project
author: shashankkatiyar
externalLink: false
---

![Gene Network](/assets/images/gene_network.png)

Professor Mohit Kumar Jolly's lab works in the field of systems biology. In this, we study biological systems through mathematical models and try to predict their behavior, with a focus on cancer disease and metastasis. My work was specifically in the field of gene networks- how the macroscopic interactions of genes among themselves create the phenotypes we observe. I used Machine Learning to predict the stability of certain gene networks.

<p>There is a class of gene networks called Toggle Polygons. Imagine a graph of nodes, in the shape of suppose, a hexagon. In a toggle polygon, the adjacent nodes of the hexagon will inhibit each other and so on. These toggle polygons are stable when the alternating nodes have high expression and others have low. I wanted to extend this knowledge and know more about what happens when we induce changes in the configuration in the topology of the toggle polygon.</p>

<p>I used to tool called RACIPE to simulate the toggle polygon stable states and automated the simulation process to get a large amount of data, to train my ML model. Later, I also introduced macroscopic variables about the network to assist the model in learning about the network better. At the end, I was able to predict the stable states of toggle polygon variants (upto those with 10 nodes) with >90% accuracy. The model can be further improved with more data and better encoding of macroscopic network parameters. This understanding about gene networks will help us to better understand the cellular mechanisms behind metastasis, in which cancer cells can change their type and spread cancer to other organs in the body.</p>
