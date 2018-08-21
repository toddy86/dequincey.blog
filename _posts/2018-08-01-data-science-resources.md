---
layout: post
title: Data Science Resources
categories: projects
---

A living and ever changing library of useful technologies, libraries and platforms for data science.

<!-- more -->

<h2>Python</h2>

<h3><a href="https://github.com/fastai/fastai">fast.ai</a></h3>
Fast.ai is a machine and deep learning library designed by Jeremy Howard with the intention to allow developers without a maths background to develop world class ML/DL models in the shortest time possible. Built on PyTorch.

Accompanying the library is the <a href="http://www.fast.ai/">fast.ai MOOC</a>, which has been the best ML / DL course I have completed to date.

The <a href="http://www.fast.ai/topics/">fast.ai blog</a> is also an invaluable resource

{% highlight ruby %}
# Installation
pip install fastai
{% endhighlight %}


<h3><a href="https://github.com/fastai/fast_progress">Fast progress</a></h3>
Simple and flexible progress bar for Jupyter Notebook and console, developed by the fast.ai team

![Example of fast progress]({{ "/assets/images/fast_progress.gif"|absolute_url}})

{% highlight ruby %}
# Installation
pip install fast_progress
{% endhighlight %}


<h3><a href="https://github.com/EpistasisLab/tpot">TPOT</a></h3>
TPOT (Tree-based Pipeline Optimization Tool) is an automated machine learning tool that optimizes machine learning pipelines using genetic programming.

TPOT is built on top of Scikit-learn and it automates the most tedious parts of machine learning like feature selection, model selection, feature construction, etc, by exploring thousands of possible pipelines to find the best one for the data. It then provides you with the Python code for the best pipeline it found for manual exploration and tweaking.

Follow the installation instructions: <a href="http://epistasislab.github.io/tpot/installing/">Installation Guide</a>

{% highlight ruby %}
# Then install TPOT
pip install tpot
{% endhighlight %}

<h3><a href="https://www.featuretools.com/">FeatureTools</a></h3>
Featuretools is a framework to perform automated feature engineering. It excels at transforming temporal and relational datasets into feature matrices for machine learning.

{% highlight ruby %}
# Installation
pip install featuretools
{% endhighlight %}
Then follow the quickstart guide: <a href="https://docs.featuretools.com/#minute-quick-start">5 minute Quick Start</a>
