---
layout: page
title: Detecting COVID-19 from Chest X-Rays
description: Using machine learning algorithms to detect COVID-19 and other respiratory illness using radiographic images.
img: assets/img/covid_xray_learn.png
importance: 3
category: grad work
github: https://github.com/rothej/jh_ml_covid_detect/
---

<div class="d-flex justify-content-center mb-4">
  <a href="https://github.com/rothej/jh_ml_covid_detect/" class="btn btn-outline-primary" target="_blank">
    <i class="fab fa-github"></i> View Code
  </a>
</div>

As my final project for my Machine Learning for Signal Processing class, I took chest x-ray images of both healthy patients and patients with COVID-19 and trained a Machine Learning algorithm to detect whether the patient had an infection based off the image alone. Additionally, I went a bit beyond the scope of the project and added in x-ray images of viral pneumonia and other respiratory infections - obviously the difference between various infections would be much harder for an algorithm to detect. Thousands of images were used at various angles and qualities, and I tested several different models such as KNN, Naive Bayes, and even some DeepLearning. The GitHub repository (which holds both the datasets and several different Matlab live scripts for various models) is linked above.

Final paper is available as a PDF.

<div class="pdf-container" style="height: 600px;">
  <embed src="{{ '/assets/pdf/covid_xray.pdf' | relative_url }}" 
         type="application/pdf" 
         width="100%" 
         height="100%">
</div>