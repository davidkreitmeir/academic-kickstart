---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Correction for Prediction Errors in Event Studies"
subtitle: ""
summary: "Short guide on how to correct for prediction errors in event studies in STATA"
authors: [David]
tags: []
categories: []
date: 2020-02-11T15:36:36+11:00
lastmod: 2020-02-11T15:36:36+11:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

In most textbooks (...) the prediction error correction for event studies in Finance is presented in matrix form. This often raises the question on how to best implement the correction in code format. Below I present a simple formula taken from (...) to 
