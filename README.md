# Hacking the Human Vasculature in 3D Competition
The 13th place solution of the SenNet + HOA Kaggle competition: ["Hacking the Human Vasculature in 3D"](https://www.kaggle.com/competitions/blood-vessel-segmentation). The goal of this competition was to segment blood vessels from 3D Hierarchical Phase-Contrast Tomography (HiP-CT) data from human kidneys. This competition was organized by Kaggle, CIFAR, ThermoFisher and others, and had a total prize pool of $80.000. My best scoring submission to this competition resulted in 13th place out of 1.149 teams and 1.401 competitors. 
![3D segmentation of Kidney_2](/assets/prediction_3Dsegmentation.gif)
## Data
TIFF CT scans of three different kidneys were available to train the machine learning models on. With each image representing a 2D slice of the whole 3D kidney volume. For each corresponding CT scan a segmentation mask was available, annotating the blood vessels, which could be used as ground truth for training. The segmentation masks of the 3 different kidneys had differences in the percentage of full segmentation, meaning in some kidney volumes blood vessel segmentations were missing.
I chose to train on the kidney_1_dense and kidney_3_dense volumes. These two kidney volumes did not have missing segmentations and therefore should in theory be better able to segments smaller blood vessel. The models were validated on the kidney_2 volume, which was missing around 35% of the blood vessels. As not all the blood vessels were manually annotated and available, this led to a validation score that did not always match the leaderboard score.

![Example of a 4-panel image and corresponding labels](/assets/example_4panel.png)

## Model
Each 2D slice of the whole 3D kidney volume was split up into patches of 256px by 256px, to keep to original resolution. Patches of subsequent slices were added into a 4-panel image, creating a 2.5D dimensionality in a 2D image. For instance, slice number 504 is selected of kidney_1_dense and subsequently split up in patches, of which patch number 5 is selected. Then, slice number 505, 506 and 507 are taken, split up in the same manner and patch number 5 of these slices is selected. All the (same) patches are added into a 4-panel 512px by 512px 2D image. The idea was that the model would learn this relationship between slices and therefore would predict a more continuous segmentation.
These 4-panel images were generated on 3 whole kidney volume rotations, with 40% overlap between patches, creating ~490.000 different training images. Image augmentation was performed on the individual 256px by 256px patches: rotate90, flipping, RandomBrightness, Blur and MotionBlur. The used segmentation model was a maxvit_tiny_tf_512 Unet using the segmentation models pytorch (SMP) library. The model was trained for 9 epochs with a learning rate (lr) of 1e-4 and subsequently another 6 epochs with CosineAnnealingLR to a lr of 1e-6.
## Submission
For the competition submission the kidney slices were split up in patches with 10% overlap. Each patch in 4-panel image was 4 times rotated on each own. Whereafter, the mean was taken of the rotated patches. The mean was taken of all the patches in the separate 4-panel images, as most patches were predicted 4 times (once for each quarter of the 4-panel). These patches were merged with the ‘max’ setting, generating a segmentation mask in the original resolution. This was performed for the xy, xz, yz rotations of the whole kidney volume.

Link to the Kaggle write up: [13th place solution: 4-panel solo model](https://www.kaggle.com/competitions/blood-vessel-segmentation/discussion/475117)
Link to the Kaggle interference notebook: [SenNet-HOA23 | 2.5D 4-panel | submission](https://www.kaggle.com/code/menno1111/sennet-hoa23-2-5d-4-panel-submission/notebook)

