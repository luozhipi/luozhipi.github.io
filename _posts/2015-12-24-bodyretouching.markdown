---
layout: post
title:  "parameterize 3D shape and semantic reshaping"
date:   2015-12-24 19:06:56
categories: animation
---

Recently, I had conversations with [Shan Yang] from UNC about the topic of `reshaping` of 3D models, actually fitting a 3D human model onto a 2D static photo.
As previously met [Yu Chen] from [metail] at a graphics conference who is researching new methods on the similar topic for virtual-try.
Thus, I then just felt this topic is quite promising and interesting to graphics community.

Here, the paper [Parametric Reshaping of Human Bodies in Images] would be a good starting point to read.

Match-and-Defom is techniqucally reasonable method. Reshape a 3D model with semantic attributes is an easier task than by low-level, local edits on a 2D image directly.
Matching by pose-driven approach can resort to skeletal rigging, and deform by the concept of `offset defomer`. Given the correspondences between the image and model contour,
body-aware image warping is then completed.

Here, as I have worked on RBF interpolation for a while. An idear came to my mind. It is, the correspondences's displacements can be the domain of RBF for the image wapring.
When reading the paper mentioned above, indeed they talked it!

[Shan Yang]: www.cs.unc.edu/~alexyang/
[metail]: http://metail.com/
[Yu Chen]: http://mi.eng.cam.ac.uk/~yc301/
[Parametric Reshaping of Human Bodies in Images]: http://sweb.cityu.edu.hk/hongbofu/projects/ParametricBodyReshaping/
