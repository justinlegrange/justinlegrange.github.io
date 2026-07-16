---
title: "KodeKloud's 100 Days of MLOps: Days 1 - 10"
date: 2026-05-11 # YYYY-MM-DD
lastMod: 2026-07-16
summary: "A walkthrough for days 1 through 10 of KodeKloud's 100 Days of MLOps challenges."
# draft: true
series: ["KodeKloud's 100 Days of MLOps"]
categories: ["DevOps", "MLOps", "KodeKloud"]
tags: ["devops", "mlops", "kodekloud", "llms", "python"]
---

## Intro

Welcome to a new series of blog posts centered around the [KodeKloud "100 Days of MLOps" challenge](https://kodekloud.com/100-days-of-mlops)! If you've explored the blog before, the `100 Days of X` challenges that KodeKloud puts out are a great set of exercises, in this case centered around MLOps! I've been really enjoying the 100 Days of DevOps series of challenges, and so I imagine that I'll love digging in to the MLOps side of the house as well. Hopefully you enjoy it as much as I do!

> [!IMPORTANT]+ Spoiler alert!
> In case you're squeamish about this sort of thing, there are a bunch of spoilers ahead - proceed at your own (self-learning) risk. I'll be diving into the nitty-gritty behind solutions where I can, so hopefully you'll be able to learn a thing or two.  
> 
> It's also worth noting that if you're working alongside me, you'll see different users, IP addresses, passwords, or even completely different solutions occasionally - they rotate these with each challenge spawn on most challenges.
{icon="circle-info"}

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

And that's it - a functional `venv` set up with the necessary packages.

## Day 2: Set Up and Configure Jupyter Notebook Server

Placeholder.

## Day 3: Fix a Broken uv Lockfile Specification

Placeholder.

## Day 4: Create a Standard ML Project Structure

Placeholder.

## Day 5: Create a Makefile for ML Workflow Automation

Placeholder.

## Day 6: Set Up Code Quality Tools for ML Code

Placeholder.

## Day 7: Package an ML Project as Installable Python Package

Placeholder.

## Day 8: Configure Pre-Commit Hooks for ML Repository

Placeholder.

## Day 9: Create a Custom ML Project Template with Cookiecutter

Placeholder.

## Day 10: Install and Initialize DVC in an ML Project

Placeholder.
