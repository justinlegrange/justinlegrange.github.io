---
title: "KodeKloud's 100 Days of MLOps: Days 1 - 10"
date: 2025-05-11 # YYYY-MM-DD
description: "A walkthrough for days 1 through 10 of KodeKloud's 100 Days of DevOps challenges."
# weight: 1
# aliases: ["/first"]
draft: true
series: ["KodeKloud's 100 Days of MLOps"]
categories: ["MLOps", "KodeKloud", "LLMs"]
tags: ["devops", "mlops", "kodekloud", "llms", "python"]
showToc: true
TocOpen: false
hidemeta: false
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: true
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
    cover.responsiveImages: true
---

## Intro

Welcome to a new series of blog posts centered around the [KodeKloud "100 Days of MLOps" challenge](https://kodekloud.com/100-days-of-mlops)!

## Day 1: Create a Python Virtual Environment for ML

> [!QUOTE] Problem Prompt
> The xFusionCorp Industries data science team needs a standardised Python environment for their new ML project. Set up a virtual environment with the required ML libraries on the controlplane host.
> 
> Create a Python virtual environment named ml-env under /root/code/ using python3 -m venv.
> Activate the environment and install the following packages: numpy, pandas, scikit-learn, and matplotlib.
> Generate a requirements.txt file using pip freeze and save it at /root/code/requirements.txt.
{icon="circle-question"}

This is a really straightforward set of tasks, especially if you're familiar with Python to any degree. You've likely used one of the virtual environments before to keep dependencies straight and remove the risk of package conflicts across different projects. It's nice and clean, super easy to use - there's no reason to not deploy `venv` on a project, really.

To set up the environment, create a new one with the `venv` module (first command) and then activate it, dropping into our clean-slate environment (second command):

```bash
$ python3 -m venv ml-env
$ source ml-env/bin/activate
```

If you've used Python for any sort of development before, this workflow should feel familiar - install the libraries using `pip install` and then lock the versions using `pip freeze` like so:
```bash
$ pip install numpy pandas scikit-learn matplotlib
$ pip freeze > requirements.txt
```