---
layout: post
title: '30 FPS Body Pose Estimation on mobile using a spatio-temporal CNN-based model on RGB and depth (ToF) images'
description: 'A preview of the capabilities of my models for Human Pose Estimation. The models are based on ConvLSTMs and ConvGRUs. They are running in real-time on mobile devices.'
tags: [CV, DeepLearning, HPE, Tensorflow, Python]
date-string: MARCH 18, 2019
image:
  feature: 2019-03-18/BodyPoseEstimationHeader.png
  show-at-top: false
external-links: 
  github: tsucres/BodyPoseEstimation
---

This is a preview of the results (description of the architectures + code are coming soon). The footages come from the NTU RGB+D Action Recognition Dataset [1].

(The gifs link to a video of better quality)

[![](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/victory_rgb_gru.gif)](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/victory_rgb_gru.mov)

[![](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/victory_rgb_gru2.gif)](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/victory_rgb_gru2.mov)


#### ConvLSTM model on RGB

[![](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/shoeoff_rgb_lstm.gif)](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/shoeoff_rgb_lstm.mov)

#### ConvGRU model on depth


[![](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/hat_depth_gru.gif)](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/hat_depth_gru.mov)

[![](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/glasseson_depth_gru.gif)](https://github.com/tsucres/BodyPoseEstimation/raw/90e778093ebd0b775c72254d0e09723c02b72990/assets/glasseson_depth_gru.mov)


### References

[1] Shahroudy, A., Liu, J., Ng, T.-T., & Wang, G. (2016). NTU RGB+D: A Large Scale Dataset for 3D Human Activity Analysis. In The IEEE Conference on Computer Vision and Pattern Recognition (CVPR).