---
title: ":smoking: Inferring Evolution of Oral Cancer"
layout: post
date: 2022-11-16 22:10
tag: 
- Genome Analysis
- Bioinformatics
image: assets/images/intratumour_het.png
# headerImage: true
projects: true
hidden: true # don't count this post in blog pagination
description: "This is description of my work in identifying intratumour heterogeneity in oral cancer patients."
category: project
author: shashankkatiyar
externalLink: false
---

![Intratumour Heterogeneity](/assets/images/intratumour_het.png)

While at IIT Kanpur, I worked under Professor Hamim Zafar, who works in the field of genomics. Due to widespread tobacco consumption in India, Oral cancer cases are really common there. He gave me a task to characterize intra-tumour heterogeneity in oral cancer patients, through phylogenetics which would help doctors decide on the specific drugs to be given to a particular patient.

<p>I went on to find existing tools that could do this. I decided to use the DeCiFer tool. It takes in information about point mutations and Copy Number Variations in the patient’s cancer and develops the phylogeny of the intra-tumor heterogeneity of the cancer. However, I needed to extract the VCF and copy the number variation information before. For VCF, I used Mutect2 by GATK (genome analysis toolkit by Broad Institute). However, for CNVs all the programs I tried were outdated or not functional on MacOS. Eventually, I stumbled across the ABSOLUTE tool, which although is written in R (my whole project otherwise was on Python), worked well. Finally, I was able to use this information to develop the phylogeny of the intra-tumor heterogeneity of oral cancer patients using DeCiFer.</p>

<p>The phylogeny of intra-tumor heterogeneity in an oral cancer patient will help better categorize it and help doctors prescribe more specific drugs, which will target cancer cells better and affect normal cells less. This will pave the way for a reliable cancer cure.</p>
