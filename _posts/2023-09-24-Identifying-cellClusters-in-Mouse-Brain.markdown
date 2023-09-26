---
title: "🧠 Clustering cells from Mouse Brain"
layout: post
date: 2023-04-01 22:10
tag: 
- Deep Learning
- Autoencoder
- Clustering
image: assets/images/mouse_brain_clustering.png
# headerImage: true
projects: true
hidden: true # don't count this post in blog pagination
description: "Description of my project on clustering cells from Mouse Brain"
category: project
author: shashankkatiyar
externalLink: false
---

![Mouse Brain Clustered](/assets/images/mouse_brain_clustering.png)

In this project, I had to cluster the cells from the mouse brain. Given was the real clustering data from one tissue layer of the mouse and using the gene expression information from the spots in that, I had to predict the cells for another layer. The data from obtained using the Visium technology.

<p>I used an autoencoder (from scvi-tools) to encode the gene expression information into a lower dimensional space and then used the encoded information to cluster the cells. The autoencoder was trained on the real data and then used to predict the cells for the other layer. The results were pretty good, as shown in the figure.</p>


