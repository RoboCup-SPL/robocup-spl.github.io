---
title: Open Source
layout: default
---

TODO: transfer contents of https://spl.robocup.org/open-source/
TODO: add links and descriptions

## Full Code Releases

Code Releases that can be deployed to robots

- Berlin United
- B-Human
- HULKs
- Nao Devils Dortmund
- rUNSWift
- UT Austin Villa
- Northern Bites (deprecated?)
- UPennalizers (from 2016, deprecated?)
- ...

## Major Component Releases

- Bembelbots particle filter
- HTWK stuff
- SPQR Ball Perceptor
- UChile Ball Perceptor
- TJArk Vision 2018
- TJArk 2019
- ...

## Tools

- HTWK LolaConnector
- Bembelbots WebotsLoLaController
- ...

## Neural Network Inference

- Nao Devils Dortmund: [nncg](https://github.com/iml130/nncg) ([Paper](https://arxiv.org/abs/2001.05572))
- B-Human: [CompiledNN](https://github.com/bhuman/CompiledNN) ([Paper](https://b-human.de/downloads/publications/2019/CompiledNN.pdf))
  - I recently (04/2021) benchmarked it against TensorFlow Lite with XNNPACK and found out that the performance gap to TensorFlow Lite is not that large anymore. Due to this and the number of bugs/inaccuracies/unsupported features in CompiledNN, we only recommend CompiledNN for small classifier networks that are executed many times per image. For larger networks that are only executed once per image, the performance advantage of CompiledNN may not be large enough (and in some cases it might even be at disadvantage to TensorFlow Lite).
- HTWK Robots: [caffe](https://github.com/tkalbitz/caffe/tree/nao-optimized-caffe)

## System Software

- Nao Devils Dortmund: custom Ubuntu OPN generator (ask someone for a link, not sure yet whether it may be published)
- SoftBank Robotics: [the original 4.4-rt kernel that is used in the 2.8.5 OPNs](https://github.com/aldebaran/linux-aldebaran/tree/sbr/v4.4.86-rt99-baytrail)
- B-Human: [patched 5.4-rt kernel](https://github.com/bhuman/Kernel)
