## 3D human reconstruction from 2D video

Approach - [PIFuHD: Multi-level Pixel-Aligned Implicit Function for High-Resoltuion 3D human digitization](https://arxiv.org/abs/2004.00452)

I have chosen this approach becasue PiFuHD is trained on high-resolution(1k) images to achieve high-fidelity 3D reconstructions from a single image that capture detailed information such as fingers, hair, facial features, clothing. Though the model fails in certain scenarios, it does a good job at capturing the fine details in the cases where it does well.

### PIFuHD overview:
![image](https://user-images.githubusercontent.com/14092419/149799708-80205899-9542-4135-8bfc-0f20d6cada3f.png)

PIFuHD is an end-to-end trainable multi-level coarse-to-fine framework that infers 3D geometry in a pixel-aligned manner and retains the details in the original inputs without any post processing. It builds on the PIFu framework and can also effectively handle uncertainty in unobserved regions.

PIFuHD stacks an additional pixel-aligned prediction module on top of PIFu(512x512 input images) to achieve higher resolution outputs. This fine module takes higher resolution (1024x1024) images and encodes into high resolution image features (512x512). The module then uses this high resolution feature embedding and the 3D embeddings from the coarse PIFu module to predict an occupancy probability field. Normal maps for the front and back sides are predicted in the image space and fed to the network as additional inputs to improve the quality and fidelity of the reconstruction.

The coarse PIFu module captures global 3D structure - takes the downsampled 512x512 image and produces backbone image features of 128x128 resolution.
The fine level modules takes the original 1024x1024 image and produces 512x512 backbone image features. The network uses an extended binary Cross Entropy loss at a set of sampled points.

Instead of inferring the backside entirely using the MLP prediction network (which causes smooth and featureless 3D reconstructions), part of the inference is shifted to the feature extraction stage. Normal maps are predicted as a proxy for 3D geometry in the image space and fed as features to the pixel-aligned predictors. This makes it easier for the MLP to produce details.

### Sample Results
![image](https://user-images.githubusercontent.com/14092419/149817003-12631698-5f68-4f71-965b-ad79e4d486f3.png) ![image](https://user-images.githubusercontent.com/14092419/149817017-41155b19-5708-4eb0-a7bc-bfe3b8f21f15.png) ![image](https://user-images.githubusercontent.com/14092419/149817453-bde444b0-a1c1-4278-8f1e-c9a208fa62fa.png) ![image](https://user-images.githubusercontent.com/14092419/149817466-afe2108f-8e73-48cb-981a-0f9f2d7e9df6.png)

Motion blur in the image for JumpingJacks causes incomplete reconstructions and artifacts




All the generated OBJs for the input videos can be found [here](https://drive.google.com/drive/folders/11O-LDoiDmVR9TEC0vlGlrQTToo3Ga102?usp=sharing)

### Limitations

1. The model is trained with limited variation in data and only works well for fashion poses (front facing poses)

2. Reconstruction quality for multi-person images is poor

![image](https://user-images.githubusercontent.com/14092419/149805732-257edaad-1b06-4eae-8524-11c968248087.png) ![image](https://user-images.githubusercontent.com/14092419/149805744-0483e254-4f57-4ef1-a2f2-be8d911a4f54.png)

3. Does not work with cluttered background

These limitations can be mitigated by (a) Training the model with use-case specific images or videos from YouTube to increase the variation in training data
(b) Using a multi-person keypoint detector to obtain pose and keypoints for multiperson images to improve multi-person reconstruction (c) Using segmentation to sperate background and foreground entities

### Test suite for video converter
I would prefer having a test suite that validates the output meshes in the image space i.e. render the output objs, one per frame, and run image similarity between the output render and the original input frame. This method can also be used to train the model - use the difference between the output and the input images as an auxiliary loss during training to reduce the difference between the two.

### Improvements
Both the proposed improvements are meant as a pre-processing step - (a) keypoint detection (b) background segmentation (c) Mesh cleaning to remove artifacts
Another improvement could be to use [PyMAF: 3D Human Pose and Shape Regression with Pyramidal Mesh Alignment Feedback Loop](https://arxiv.org/abs/2103.16507) to get the human-pose and use this with PIFuHD.

### Running the code
I've used Colab for this assignment. The video is processed using OpenCV to get frames, keypoints are detected for each frame and saved in a corresponding txt file, and each image and its corresponding rectange file is fed as input to the pre-trained PIFu model. The resulting output OBJs are saved results/

