---
layout: post
title:  "nice survey papers on deformation"
date:   2015-06-30 12:13:56
categories: animation
---

Recently, I had conversations with [Shan Yang] from UNC about the topic of `reshaping` of 3D models, actually fitting a 3D human model to a 2D stiatic photo.
As previously met [Yu Chen] from [metail] at a graphics conference who is researching new methods on the similar topic for virtual-try.
Thus, I then just felt this topic is quite promising an interesting to graphics community.

Here, the paper [Parametric Reshaping of Human Bodies in Images] would be a good starting point to read.

Match-and-Defom is techniqucally reasonable method. reshape a 3D model with semantic attributes is a easier task than by low-level, local edits on a 2D image.
Matching by pose-driven approach can resourt to skeletal rigging, and deform by `offset defomer`. Given the correspondences between the image contour and model contour,
body-aware image warping is then completed.

Here, as I have worked on RBF interpolation for a while. An idear came to my mind. It is, the correspondences's displacements can be the domain of RBF for the image wapring.
When reading the paper mentioned above, indeed they talked it!

[Shan Yang]: www.cs.unc.edu/~alexyang/
[metail]: http://metail.com/
[Yu Chen]: http://mi.eng.cam.ac.uk/~yc301/
[Parametric Reshaping of Human Bodies in Images]: http://sweb.cityu.edu.hk/hongbofu/projects/ParametricBodyReshaping/
