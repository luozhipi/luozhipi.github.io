---
layout: post
title:  "skinning mesh deformations"
date:   2015-06-18 11:15:14
categories: animation
---
Machine learning could also be used in animation? Of course the anwser is a big `YES`. Notabely, used by the research of `skinning mesh deformations`, pioneered by `Doug L. James` and `Christopher D. Twigg`, see their paper entitled `Skinning Mesh Animations` presented in SIGGRAPH 2005. Later, `Ladislav Kavan` has proposed several methods of extracting the LBS model from a mesh sequence. However, they are again not suitable for rigging, as the bones transformation are not rigid.

Recent greate breakthrough is achieved by `Binh Huy Le` and `Zhigang Deng`. They have extended to rigid bones, common to motion editing and skeleton extraction. Succeeded by their work presented in SIGGRAPH 2014, their `skinning decomposition` method now is largely improved, making it very promising to converting deformations as artist-friendly rigging.

In addition, skinning decomposition has another notable metrits: data compression. A LBS model of course meeds much lower storage than mesh sequences, in particult for blendshapes model which is still the common solution for facial animation.

But, have to say, extracting a LBS model is limited to the input data which contains noisy without a doublt. This is because the examplar poses are often do not exist. They are obtained by handcraft or capturing.

Let's say, in facial animation, seconday deformations are very neccesary to provide the realism. However, LBS is not efficient to reproduce such visual effects, in turn, it is of course not possible to learn a LBS model that will reproduce the deformations. So, we need to smooth out secondary deformations untile we obtain the deformation details that can be reached by linear blend skinning. This will be very empirical.

`Webber Huang` [[demo]] has made a greate demo, showing that using `Delta Mush`, a new smoothing techniques that will be included in MAYA 2016, combined with `skinning decomposition with rigid bones`, has largely improved the performance, both in character deofmraiton, and facial animation.

[demo]:      http://riggingtd.com/2015/06/deformation-learning-solver/
